---
description: Creating a OAuth2 enabled resource server.
icon: arrow-right-arrow-left
---

# Backend Token Validation

Now that you have setup oidc-spa in your web project you can make calls like this: &#x20;

```typescript
const todos = fetch("/api/todos", { 
    headers: {
        Authorization: `Bearer ${await oidc.getAccessToken()}`
    }
});
```

Now we're going to see how to implement the backend side fo things. &#x20;

When you implement the server GET /todos endpoint handler you want to be able to read the Authorization header of the request to establish and validate the identity of the user and optionally check if that user have sufisent permissions to perform this request. &#x20;

If your are implementing a JavaScript backend, for example Express, Hono, tRPC, Nest.js ect, oidc-spa provide the tools you need to validate and decode the access token. The validation process inclues DPoP proof check and replay protection. &#x20;

<details>

<summary>More context</summary>

The server side validation utils that oidc-spa offers implements [RFC 9068: JSON Web Token (JWT) Profile for OAuth 2.0 Access Tokens](https://datatracker.ietf.org/doc/rfc9068/).&#x20;

The great thing about JWT validation is that it works offline, there’s no need to contact your authorization server every time to ask “Is this token valid and issued by you?”

oidc-spa (server) simply fetches the public key published by your IdP once, then uses it to verify that each incoming token:&#x20;

* was signed by the IdP,&#x20;
* targets the expected audience&#x20;
* hasn’t expired and,
* validate [DPoP proof](../features/dpop.md) (if applicable)

This is a huge advantage for edge runtimes, since identity and authorization can be established locally no external round trips before executing user-specific logic.

To authorize certain routes or actions, you can perform additional checks on claims like `groups` or `realm_access.roles`.

Note however that some IdP do not issue JWT access token by default yet. Some still issue opaque access tokens. Opaque access tokens cannot be validated in a provider agnostic way like JWT can. If your IdP issue opaque access token you'll need to use their specific tooling for validating token and won't be able to use oidc-spa/server. &#x20;

</details>

## Integration

Integration example with popular API framworks.&#x20;

{% tabs %}
{% tab title="Hono" %}
Export your Authentication and Authorization utils:

{% code title="src/auth.ts" %}
```typescript
import { oidcSpa } from "oidc-spa/server";
import { z } from "zod";
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

// This is your internal abstraction of an user
export type User = {
    id: string;
};

export async function getUser(
    req: HonoRequest,
    requiredRole?: "realm-admin" | "support-staff"
): Promise<User> {

    // NOTE: You can also do `validateAndDecodeAccessToken({ accessToken })`, 
    // it just won't work with DPoP bound access tokens.  
    const { isSuccess, errorCause, debugErrorMessage, decodedAccessToken } =
        await validateAndDecodeAccessToken({
            request: {
                url: req.url,
                method: req.method,
                getHeaderValue: headerName => req.header(headerName)
            }
        });

    if (!isSuccess) {

        if( errorCause === "missing Authorization header" ){
            // Demo shortcut: we return 401 on missing Authorization, but a mixed
            // public/private endpoint could instead return undefined here and let
            // the caller decide whether to process an anonymous request.
            console.warn("Anonymous request");
        }else{
            console.warn(debugErrorMessage);
        }

        throw new HTTPException(401); // Unauthorized
    }

    if (
        requiredRole !== undefined &&
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
```
{% endcode %}

Using your utils:

<pre class="language-typescript" data-title="src/main.ts"><code class="lang-typescript">
import { Hono } from "hono";
import * as fs from "node:fs/promises";
<strong>import { bootstrapAuth, getUser } from "./auth";
</strong>
function startHonoServer() {

<strong>    bootstrapAuth({
</strong><strong>        implementation: "real", // or "mock"
</strong><strong>        issuerUri: process.env.OIDC_ISSUER_URI!,
</strong><strong>        expectedAudience: process.env.OIDC_AUDIENCE
</strong><strong>    });
</strong>
    const app = new Hono();

    app.get("/api/todos", async c => {

<strong>        const user = await getUser(c.req);
</strong>
        const json = await fs.readFile(`todos_${user.id}.json`, "utf8");

        return c.text(json);

    });

    // ...

}
</code></pre>
{% endtab %}

{% tab title="Express" %}

{% endtab %}

{% tab title="tRPC" %}

{% endtab %}

{% tab title="Nest.js" %}
For Nest.js there is a comunity wrapper around `oidc-spa/server`:

{% embed url="https://github.com/mwolf1989/nestjs-spa-oidc" %}
{% endtab %}

{% tab title="TanStack Start" %}

{% endtab %}
{% endtabs %}

## TODO List Example

Here is a TODO list application example build with Vite / React / TanStack Router for the frontend and the backend REST API build with Node.js / Hono.

{% embed url="https://youtu.be/33VijFArY9s" %}

The app is live here:

{% embed url="https://vite-insee-starter.demo-domain.ovh/" %}

The source code of the REST API:

{% embed url="https://github.com/InseeFrLab/todo-rest-api" %}

The source code of the frontent:

{% embed url="https://github.com/InseeFrLab/vite-insee-starter" %}
