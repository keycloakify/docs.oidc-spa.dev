---
icon: fire
---

# Hono

This is how your API handler would typically look like:

{% code title="src/main.ts" %}
```ts
import { Hono } from "hono";
import * as fs from "node:fs/promises";
import { bootstrapAuth, getUser } from "./auth"; // See below

function startHonoServer() {

    bootstrapAuth({
        implementation: "real", // or "mock"
        issuerUri: process.env.OIDC_ISSUER_URI!,
        expectedAudience: process.env.OIDC_AUDIENCE
    });

    const app = new Hono();

    app.get("/api/todos", async c => {

        const user = await getUser(c.req);

        const json = await fs.readFile(`todos_${user.id}.json`, "utf8");

        return c.text(json);

    });

    // ...

}
```
{% endcode %}

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
    req: HonoRequest,
    requiredRole?: "realm-admin" | "support-staff"
): Promise<User> {
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

    const user: User = {
        id: decodedAccessToken.sub
    };

    return user;
}
```
{% endcode %}
