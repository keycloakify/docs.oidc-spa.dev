---
icon: leaf
---

# Koa

This is how your API handler would typically look like:

<pre class="language-ts" data-title="src/main.ts"><code class="lang-ts">import Koa from "koa";
import Router from "@koa/router";
import * as fs from "node:fs/promises";
<strong>import { bootstrapAuth, getUser } from "./auth"; // See below
</strong>
async function startKoaServer() {

<strong>    bootstrapAuth({
</strong><strong>        implementation: "real", // or "mock", see: https://docs.oidc-spa.dev/v/v8/integration-guides/backend-token-validation/mock-modes
</strong><strong>        issuerUri: process.env.OIDC_ISSUER_URI!,
</strong><strong>        expectedAudience: process.env.OIDC_AUDIENCE
</strong><strong>    });
</strong>
    const app = new Koa();

    const router = new Router();

    router.get("/api/todos", async ctx => {

<strong>        const user = await getUser({ ctx });
</strong>
        const json = await fs.readFile(
<strong>            `todos_${user.id}.json`,
</strong>            "utf8"
        );

        ctx.status = 200;
        ctx.type = "application/json";
        ctx.body = json;

    });

    router.get("/api/todos-for-support/:userId", async ctx => {

        // Will reject the request if user making the request
        // doesn't have "support-staff" role
<strong>        await getUser({ ctx, requiredRole: "support-staff" });
</strong>
        const json = await fs.readFile(
<strong>            `todos_${ctx.params.userId}.json`,
</strong>            "utf8"
        );

        ctx.status = 200;
        ctx.type = "application/json";
        ctx.body = json;

    });

    app.use(router.routes());
    app.use(router.allowedMethods());

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
import type { Context } from "koa";

const { bootstrapAuth, validateAndDecodeAccessToken } = oidcSpa
    .withExpectedDecodedAccessTokenShape({
        // This is purely declarative. Here you'll specify
        // the claim that you expect to be present in the access token payload.
        decodedAccessTokenSchema: z.object({
            sub: z.string(),
            // Keycloak specific, convention to manage authorization.
            realm_access: z
                .object({
                    roles: z.array(z.string())
                })
                .optional()
        })
    })
    .createUtils();

export { bootstrapAuth };

// Your local representation of a user.
export type User = {
    id: string;
};

export async function getUser(params: {
    ctx: Context;
    requiredRole?: "realm-admin" | "support-staff";
}): Promise<User> {

    const { ctx, requiredRole } = params;

    const requestAuthContext = extractRequestAuthContext({
        request: ctx.req,
        // Set this to false only if you don't have a reverse HTTP proxy in front of your
        // server. (Almost never the case in modern deployments).
        trustProxy: true
    });

    if (!requestAuthContext) {
        console.warn("Anonymous request");
        ctx.throw(401); // Unauthorized
    }

    if (!requestAuthContext.isWellFormed) {
        console.warn(requestAuthContext.debugErrorMessage);
        ctx.throw(400); // Bad Request
    }

    const { isSuccess, debugErrorMessage, decodedAccessToken } =
        await validateAndDecodeAccessToken(
            requestAuthContext.accessTokenAndMetadata
        );

    if (!isSuccess) {
        console.warn(debugErrorMessage);
        ctx.throw(401); // Unauthorized
    }

    if (requiredRole) {
        if (!decodedAccessToken.realm_access?.roles.includes(requiredRole)) {
            console.warn(`User missing role: ${requiredRole}`);
            ctx.throw(403); // Forbidden
        }
    }

    return { id: decodedAccessToken.sub };
}
```
{% endcode %}
