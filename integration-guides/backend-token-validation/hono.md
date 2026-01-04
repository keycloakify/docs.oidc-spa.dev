---
icon: fire
metaLinks:
  alternates:
    - /broken/spaces/UhNOMoIddws1XoAnT5Nn/pages/ouqjtmsZVJywv5rMklIc
---

# Hono

This is how your API handler would typically look like:

<pre class="language-ts" data-title="src/main.ts"><code class="lang-ts">import { Hono } from "hono";
import * as fs from "node:fs/promises";
<strong>import { bootstrapAuth, getUser } from "./auth"; // See below
</strong>
function startHonoServer() {

<strong>    bootstrapAuth({
</strong><strong>        implementation: "real", // or "mock", see: https://docs.oidc-spa.dev/v/v8/integration-guides/backend-token-validation/mock-modes
</strong><strong>        issuerUri: process.env.OIDC_ISSUER_URI!,
</strong><strong>        expectedAudience: process.env.OIDC_AUDIENCE
</strong><strong>    });
</strong>
    const app = new Hono();

    app.get("/api/todos", async c => {

<strong>        const user = await getUser({ req: c.req });
</strong>
        const json = await fs.readFile(`todos_${user.id}.json`, "utf8");

        return c.text(json);

    });

    app.get("/api/todos-for-support/:userId", async c => {

        // Will reject the request if user making the request
        // doesn't have "support-staff" role
<strong>        await getUser({ req: c.req, requiredRole: "support-staff" });
</strong>
        const userId = c.req.param("userId");

        const json = await fs.readFile(`todos_${userId}.json`, "utf8");

        return c.text(json);

    });

    // ...

}
</code></pre>

Let's see how to export the utils to make it happen:

{% code title="src/auth.ts" %}
```ts
import { oidcSpa, extractRequestAuthContext } from "oidc-spa/server";
import { z } from "zod";
import { HTTPException } from "hono/http-exception";
import type { HonoRequest } from "hono";

const { bootstrapAuth, validateAndDecodeAccessToken } = oidcSpa
    .withExpectedDecodedAccessTokenShape({
        // This is purely declarative. Here you'll specify
        // the claim that you expect to be present in the access token payload.
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
    req: HonoRequest;
    requiredRole?: "realm-admin" | "support-staff";
}): Promise<User> {

    const { req, requiredRole } = params;

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
        throw new HTTPException(401); // Unauthorized
    }

    if (!requestAuthContext.isWellFormed) {
        console.warn(requestAuthContext.debugErrorMessage);
        throw new HTTPException(400); // Bad Request
    }

    const { isSuccess, debugErrorMessage, decodedAccessToken } =
        await validateAndDecodeAccessToken(
            requestAuthContext.accessTokenAndMetadata
        );

    if (!isSuccess) {
        console.warn(debugErrorMessage);
        throw new HTTPException(401); // Unauthorized
    }

    // Your custom Authorization logic: Grant per request access depending
    // on the access token claim.
    if (requiredRole) {
        if (!decodedAccessToken.realm_access?.roles.includes(requiredRole)) {
            console.warn(`User missing role: ${requiredRole}`);
            throw new HTTPException(403); // Forbidden
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
