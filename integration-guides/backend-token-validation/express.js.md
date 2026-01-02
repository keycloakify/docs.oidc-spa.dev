---
icon: e
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/UhNOMoIddws1XoAnT5Nn/integration-guides/backend-token-validation/express.js
---

# Express.js

This is how your API handler would typically look like:

<pre class="language-ts" data-title="src/main.ts"><code class="lang-ts">import express from "express";
import * as fs from "node:fs/promises";
<strong>import { bootstrapAuth, getUser } from "./auth"; // See below
</strong>
function startExpressServer() {

<strong>    bootstrapAuth({
</strong><strong>        implementation: "real", // or "mock", see: https://docs.oidc-spa.dev/v/v8/integration-guides/backend-token-validation/mock-modes
</strong><strong>        issuerUri: process.env.OIDC_ISSUER_URI!,
</strong><strong>        expectedAudience: process.env.OIDC_AUDIENCE
</strong><strong>    });
</strong>
    const app = express();

    app.get("/api/todos", async (req, res) => {

<strong>        const user = await getUser({ req, res });
</strong>
        const json = await fs.readFile(
<strong>            `todos_${user.id}.json`, 
</strong>            "utf8"
        );

        res.status(200).type("application/json").send(json);

    });

    app.get("/api/todos-for-support/:userId", async (req, res) => {

        // Will reject the request if user making the request
        // doesn't have "support-staff" role
<strong>        await getUser({ req, res, requiredRole: "support-staff" });
</strong>
        const json = await fs.readFile(
<strong>            `todos_${req.params.userId}.json`,
</strong>            "utf8"
        );

        res.status(200).type("application/json").send(json);

    });

    // ...

    app.listen(parseInt(process.env.PORT ?? "3000"), () => {
        console.log("Server running");
    });
}
</code></pre>

Let's see how to export the utils to make it happen:

{% code title="src/auth.ts" %}
```ts
import { oidcSpa, extractRequestAuthContext } from "oidc-spa/server";
import { z } from "zod";
import type { Request, Response } from "express";

const { bootstrapAuth, validateAndDecodeAccessToken } = oidcSpa
    .withExpectedDecodedAccessTokenShape({
        // Here you specify the claim you expect to be present in the decoded
        // JWT payload of the access token.  
        // What's included in the token is configured on the IdP side.
        // Here you declare what your application actually uses so that
        // the type get propagated and you get a clear error if the IdP does
        // not issue what your app expects.
        decodedAccessTokenSchema: z.object({
            sub: z.string(),
            name: z.string(),
            email: z.string().optional(),
            // Keycloak specific, convention to manage authorization.
            realm_access: z.object({
                roles: z.array(z.string())
            }).optional()
        })
    })
    .createUtils();

export { bootstrapAuth };

// Your local representation of a user.
export type User = {
    id: string;
    name: string;
    email: string | undefined;
};

export async function getUser(params: {
    req: Request;
    res: Response;
    requiredRole?: "realm-admin" | "support-staff";
}): Promise<User | never> {

    const { req, res, requiredRole } = params;

    const bail = (statusCode: 400 | 401 | 403) => {
        res.sendStatus(statusCode);
        return new Promise<never>(() => {});
    };

    const requestAuthContext = extractRequestAuthContext({
        request: req,
        // Set this to false only if you don't have a reverse HTTP proxy in front of your
        // server. (Almost never the case in modern deployments).
        trustProxy: true
    });

    if( !requestAuthContext ){
        // Demo shortcut: we return 401 on missing Authorization, but a mixed
        // public/private endpoint could instead return undefined here and let
        // the caller decide whether to process an anonymous request.
        console.warn("Anonymous request");
        return bail(401); // Unauthorized
    }

    if (!requestAuthContext.isWellFormed) {
        console.warn(requestAuthContext.debugErrorMessage);
        return bail(400); // Bad Request
    }

    const { isSuccess, debugErrorMessage, decodedAccessToken } =
        await validateAndDecodeAccessToken(
            requestAuthContext.accessTokenAndMetadata
        );

    if (!isSuccess) {
        console.warn(debugErrorMessage);
        return bail(401); // Unauthorized
    }

    // Your custom Authorization logic: Grant per request access depending
    // on the access token claim.
    if (requiredRole) {
        if (!decodedAccessToken.realm_access?.roles.includes(requiredRole)) {
            console.warn(`User missing role: ${requiredRole}`);
            return bail(403); // Forbidden
        }
    }

    // Here you can potentially enrich the user object with additional
    // data that you would retrieve from your database if the access token
    // claim does not contain everything you need.

    const { sub, name, email } = decodedAccessToken;

    const user: User = { id: sub, name, email };

    return user;
}
```
{% endcode %}
