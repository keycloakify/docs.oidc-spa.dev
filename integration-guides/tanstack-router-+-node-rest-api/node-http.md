---
icon: node-js
---

# node:http

This is how your API handler would typically look like:

<pre class="language-ts" data-title="src/main.ts"><code class="lang-ts">import { createServer } from "node:http";
import { parse } from "node:url";
import * as fs from "node:fs/promises";
<strong>import { bootstrapAuth, getUser } from "./auth"; // See below
</strong>
function startNodeServer() {

<strong>    bootstrapAuth({
</strong><strong>        implementation: "real", // or "mock"
</strong><strong>        issuerUri: process.env.OIDC_ISSUER_URI!,
</strong><strong>        expectedAudience: process.env.OIDC_AUDIENCE
</strong><strong>    });
</strong>
    const server = createServer(async (req, res) => {

        const { pathname } = parse(req.url!, true);

        if (req.method === "GET" &#x26;&#x26; pathname === "/api/todos") {

<strong>            const user = await getUser(req, res);
</strong>
            const json = await fs.readFile(
<strong>                `todos_${user.id}.json`,
</strong>                "utf8"
            );

            res.writeHead(200, { "Content-Type": "application/json" });
            res.end(json);

            return;

        }

        if (
            req.method === "GET" &#x26;&#x26;
            pathname?.startsWith("/api/todos-for-support/")
        ) {

<strong>            // Will reject the request if user making the request
</strong><strong>            // doesn't have "support-staff" role
</strong><strong>            await getUser(req, res, "support-staff");
</strong>
            const userId = decodeURIComponent(
                pathname.replace("/api/todos-for-support/", "")
            );

            const json = await fs.readFile(
                `todos_${userId}.json`,
                "utf8"
            );

            res.writeHead(200, { "Content-Type": "application/json" });
            res.end(json);

            return;

        }

        res.writeHead(404).end();
    });

    server.listen(parseInt(process.env.PORT ?? "3000"), () => {
        console.log("Server running");
    });

}
</code></pre>

Let's see how to export the utils to make it happen:

{% code title="src/auth.ts" %}
```ts
import { oidcSpa, extractRequestAuthContext } from "oidc-spa/server";
import { z } from "zod";
import type { IncomingMessage } from "node:http";
import type { ServerResponse } from "node:http";

const { bootstrapAuth, validateAndDecodeAccessToken } = oidcSpa
    .withExpectedDecodedAccessTokenShape({
        // This is purely declarative. Here you'll specify
        // the claim that you expect to be present in the access token payload.
        decodedAccessTokenSchema: z.object({
            sub: z.string(),
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
};

export async function getUser(
    req: IncomingMessage,
    res: ServerResponse,
    requiredRole?: "realm-admin" | "support-staff"
): Promise<User | never> {

    const bail = (statusCode: 400 | 401 | 403) => {
        res.writeHead(statusCode).end();
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

    const user: User = {
        id: decodedAccessToken.sub
    };

    return user;
}
```
{% endcode %}
