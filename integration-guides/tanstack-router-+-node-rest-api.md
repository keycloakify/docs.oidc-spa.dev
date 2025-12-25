---
description: Creating a OAuth2 enabled resource server.
icon: arrow-right-arrow-left
---

# Backend Token Validation

Now that you’ve set up oidc-spa in your web app, you can call your API like this:

```typescript
const todos = fetch("/api/todos", { 
    headers: {
        Authorization: `Bearer ${await oidc.getAccessToken()}`
    }
});
```

Next, let’s implement the backend side of things.

When you implement the server `GET /api/todos` handler, you want to read the `Authorization` header.\
Use it to authenticate the user.\
Optionally, check permissions (roles/scopes) to authorize the request.

If you’re building a JavaScript backend (Express, Hono, tRPC, NestJS, etc.), oidc-spa provides utilities to validate and decode access tokens.\
Validation includes DPoP proof checks and replay protection.

<details>

<summary>More context</summary>

The server-side validation utilities in oidc-spa implement [RFC 9068: JSON Web Token (JWT) Profile for OAuth 2.0 Access Tokens](https://datatracker.ietf.org/doc/rfc9068/).

JWT validation works offline.\
There’s no need to contact your authorization server for every request.

`oidc-spa/server` fetches the public key published by your IdP once.\
It then uses it to verify that each incoming token:

* was signed by the IdP
* targets the expected audience
* hasn’t expired
* has a valid [DPoP proof](../features/dpop.md) (if applicable)

This is a big win for edge runtimes.\
Identity and authorization can be established locally, with no external round trips.

To authorize certain routes or actions, you can perform additional checks on claims like `groups` or `realm_access.roles`.

Some IdPs don’t issue JWT access tokens by default and issue opaque access tokens instead.

Opaque access tokens can’t be validated in a provider-agnostic way like JWTs can.\
If your IdP issues opaque access tokens, you’ll need provider-specific tooling.\
In that case, you won’t be able to use `oidc-spa/server`.

</details>

## Integration

Examples for popular API frameworks.

{% tabs %}
{% tab title="Hono" %}
Export your authentication and authorization utilities:

<pre class="language-typescript" data-title="src/auth.ts"><code class="lang-typescript"><strong>import { oidcSpa } from "oidc-spa/server";
</strong>import { z } from "zod";
import { HTTPException } from "hono/http-exception";
import type { HonoRequest } from "hono";

const { bootstrapAuth, validateAndDecodeAccessToken } = oidcSpa
    .withExpectedDecodedAccessTokenShape({
        decodedAccessTokenSchema: z.object({
            sub: z.string(),
            realm_access: z.object({
                roles: z.array(z.string())
            })
        })
    })
    .createUtils();

export { bootstrapAuth };

// This is your internal abstraction of a user
export type User = {
    id: string;
};

export async function getUser(
    req: HonoRequest,
    requiredRole?: "realm-admin" | "support-staff"
): Promise&#x3C;User> {

    // NOTE: You can also do `validateAndDecodeAccessToken({ accessToken })`, 
    // it just won't work with DPoP-bound access tokens.
    const { isSuccess, errorCause, debugErrorMessage, decodedAccessToken } =
        await validateAndDecodeAccessToken({
            request: {
                url: req.url,
                method: req.method,
                getHeaderValue: headerName => req.header(headerName)
            }
        });

    if (!isSuccess) {

        if (errorCause === "missing Authorization header") {
            // Demo shortcut: we return 401 on missing Authorization, but a mixed
            // public/private endpoint could instead return undefined here and let
            // the caller decide whether to process an anonymous request.
            console.warn("Anonymous request");
        } else {
            console.warn(debugErrorMessage);
        }

        throw new HTTPException(401); // Unauthorized
    }

    if (
        requiredRole !== undefined &#x26;&#x26;
        !decodedAccessToken.realm_access.roles.includes(requiredRole)
    ) {
        console.warn(`User missing role: ${requiredRole}`);
        throw new HTTPException(403); // Forbidden
    }

    const user: User = {
        id: decodedAccessToken.sub
    };
    
    return user;
}
</code></pre>

Using your utilities:

{% code title="src/main.ts" %}
```typescript
import { Hono } from "hono";
import * as fs from "node:fs/promises";
import { bootstrapAuth, getUser } from "./auth";

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
{% endtab %}

{% tab title="Express" %}

{% endtab %}

{% tab title="tRPC" %}

{% endtab %}

{% tab title="Nest.js" %}
For NestJS, there’s a community wrapper around `oidc-spa/server`:

{% embed url="https://github.com/mwolf1989/nestjs-spa-oidc" %}
{% endtab %}

{% tab title="TanStack Start" %}
If you are in a TanStack Start project you don't need to user `oidc-spa/server` directly. `oidc-spa/react-tanstack-start` already provides the utilities to create authed server functions and REST API endpoints.

{% content-ref url="tanstack-start.md" %}
[tanstack-start.md](tanstack-start.md)
{% endcontent-ref %}
{% endtab %}
{% endtabs %}



## TODO List Example

A TODO list example app built with Vite / React / TanStack Router on the frontend, and Node.js / Hono on the backend.

{% embed url="https://youtu.be/33VijFArY9s" %}

The app is live here:

{% embed url="https://vite-insee-starter.demo-domain.ovh/" %}

Source code (REST API):

{% embed url="https://github.com/InseeFrLab/todo-rest-api" %}

Source code (frontend):

{% embed url="https://github.com/InseeFrLab/vite-insee-starter" %}
