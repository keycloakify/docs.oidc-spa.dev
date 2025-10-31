---
description: Creating a OAuth enabled resource server.
icon: arrow-right-arrow-left
---

# Creating an API Server

With **oidc-spa**, your frontend is meant to communicate with OAuth-enabled backend services, such as REST APIs, tRPC servers, or WebSocket endpoints, that accept **JSON Web Tokens (JWTs)** as access tokens.

{% hint style="success" %}
If you’re using TanStack Start, token validation is already integrated into the higher-level adapter. You can define your resource server, whether through Authenticated Server Functions or traditional REST endpoint, directly within your TanStack project.
{% endhint %}

These tokens are sent in the `Authorization` header and allow the backend to **validate**, **decode**, and **use the claims** to perform user-specific actions.

There are countless libraries for verifying JWTs, but if you’re building your backend in **JavaScript** (Node, Deno, or Web Workers), **oidc-spa** also provides a built-in utility to validate and decode access tokens issued by your client.

The great thing about JWT validation is that it works offline, there’s no need to contact your authorization server every time to ask “Is this token valid and issued by you?”

oidc-spa (server) simply fetches the public key published by your IdP once, then uses it to verify that each incoming token:\
• was signed by the IdP,\
• targets the expected audience, and\
• hasn’t expired.

This is a huge advantage for edge runtimes, since identity and authorization can be established locally no external round trips before executing user-specific logic.

To authorize certain routes or actions, you can perform additional checks on claims like `groups` or `realm_access.roles`.

And because access tokens issued through the Authorization Code Flow + PKCE are short-lived (typically 5 minutes or less), decoupling session lifetime from token validity isn’t an issue in practice.

## Example with Express and Hono

Let’s say we have a Node.js REST API built with [**Express**](https://expressjs.com/) or [**Hono**](https://hono.dev/).\
We’ll create a function that takes an access token (and optionally a required role) and performs the following checks:&#x20;

* ✅ Validates that the token was **issued by the expected IdP**.
* ✅ Ensures the token is **still valid** (not expired or tampered with).
* ✅ Confirms the **audience** matches this specific API — meaning the token was minted for it.
* ✅ Checks that the user has the **required role**, if one is specified.

If any of these conditions fail, we throw a specialized **Hono error**, which will cause the HTTP response to return **401 Unauthorized**.

If the token passes all checks, the function returns the user’s **ID** (`sub` claim), allowing the rest of your API logic to identify who made the request.

Let's create a oidc.ts file:

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

The backend Node Todos App REST API with Express and Hono:

{% embed url="https://github.com/InseeFrLab/todo-rest-api" %}
