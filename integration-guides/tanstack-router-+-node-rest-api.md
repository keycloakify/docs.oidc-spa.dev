---
description: A full-stack example covering both the backend and frontend
icon: arrow-right-arrow-left
---

# Creating an API Server

If you're implementing a JavaScript Backend (Node/Deno/webworker) `oidc-spa` also exposes an utility to help you validate and decode the access token that your client sends in the authorization header.  \
\
Let's assume we have a Node.js REST API build with Express or Hono.  \
You can create an oidc file as such:

{% code title="src/oidc.ts" %}
```typescript
import { createOidcBackend } from "oidc-spa/backend";
import { z } from "zod";
import { HTTPException } from "hono/http-exception";

const zDecodedAccessToken = z.object({
    sub: z.string(),
    aud: z.union([z.string(), z.array(z.string())]),
    realm_access: z.object({
        roles: z.array(z.string())
    })
    // Some other info you might want to read from the accessToken, example:
    // preferred_username: z.string()
});

export type DecodedAccessToken = z.infer<typeof zDecodedAccessToken>;

export async function createDecodeAccessToken(params: { 
    issuerUri: string;
    audience: string 
}) {
    const { issuerUri, audience } = params;

    const { verifyAndDecodeAccessToken } = await createOidcBackend({
        issuerUri,
        decodedAccessTokenSchema: zDecodedAccessToken
    });

    function decodeAccessToken(params: {
        authorizationHeaderValue: string | undefined;
        requiredRole?: string;
    }): DecodedAccessToken {
        const { authorizationHeaderValue, requiredRole } = params;

        if (authorizationHeaderValue === undefined) {
            throw new HTTPException(401);
        }

        const result = verifyAndDecodeAccessToken({
            accessToken: authorizationHeaderValue.replace(/^Bearer /, "")
        });

        if (!result.isValid) {
            switch (result.errorCase) {
                case "does not respect schema":
                    throw new Error(`The access token does not respect the schema ${result.errorMessage}`);
                case "invalid signature":
                case "expired":
                    throw new HTTPException(401);
            }
        }

        const { decodedAccessToken } = result;

        if (requiredRole !== undefined && !decodedAccessToken.realm_access.roles.includes(requiredRole)) {
            throw new HTTPException(401);
        }

        {
            const { aud } = decodedAccessToken;

            const aud_array = typeof aud === "string" ? [aud] : aud;

            if (!aud_array.includes(audience)) {
                throw new HTTPException(401);
            }
        }

        return decodedAccessToken;
    }

    return { decodeAccessToken };
}
```
{% endcode %}

Then you can enforce that some endpoints of your API requires the user to be authenticated, in this example we use Hono:&#x20;

<pre class="language-typescript"><code class="lang-typescript">import { z, createRoute, OpenAPIHono } from "@hono/zod-openapi";
import { serve } from "@hono/node-server"
import { HTTPException } from "hono/http-exception";
import { getUserTodoStore } from "./todo";
<strong>import { createDecodeAccessToken } from "./oidc";
</strong>
(async function main() {

<strong>    const { decodeAccessToken } = await createDecodeAccessToken({
</strong><strong>        // Here example with Keycloak but it work the same with 
</strong><strong>        // any provider as long as the access token is a JWT.
</strong><strong>        issuerUri: "https://auth.my-company.com/realms/myrealm",
</strong><strong>        audience: "account" // default audience in Keycloak
</strong><strong>    });
</strong>
    const app = new OpenAPIHono();

    {

        const route = createRoute({
            method: 'get',
            path: '/todos',
            responses: {/* ... */}
        });

        app.openapi(route, async c => {

<strong>            const decodedAccessToken = decodeAccessToken({
</strong><strong>                authorizationHeaderValue: c.req.header("Authorization")
</strong><strong>            });
</strong>
            const todos = await getUserTodoStore(decodedAccessToken.sub).getAll();

            return c.json(todos);

        });

    }

    const port = parseInt(process.env.PORT);

    serve({
        fetch: app.fetch,
        port
    })

    console.log(`\nServer running. OpenAPI documentation available at http://localhost:${port}/doc`)

})();
</code></pre>

## Testable example

This is a kitchen think example with the following stack:

* Vite
* [TanStack Router - File Based Routing](https://tanstack.com/router/latest/docs/framework/react/routing/file-based-routing)
* A Todos Rest API implemented with Node and [Hono](https://hono.dev/)

{% embed url="https://youtu.be/33VijFArY9s" %}

The app is live here:

{% embed url="https://vite-insee-starter.demo-domain.ovh/" %}

The frontend (Vite project):

{% embed url="https://github.com/InseeFrLab/vite-insee-starter" %}

The backend (Node Todos App REST API):

{% embed url="https://github.com/InseeFrLab/todo-rest-api" %}
