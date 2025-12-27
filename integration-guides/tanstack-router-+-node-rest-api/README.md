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
* has a valid [DPoP proof](../../features/dpop.md) (if applicable)

This is a big win for edge runtimes.\
Identity and authorization can be established locally, with no external round trips.

To authorize certain routes or actions, you can perform additional checks on claims like `groups` or `realm_access.roles`.

Some IdPs don’t issue JWT access tokens by default and issue opaque access tokens instead.

Opaque access tokens can’t be validated in a provider-agnostic way like JWTs can.\
If your IdP issues opaque access tokens, you’ll need provider-specific tooling.\
In that case, you won’t be able to use `oidc-spa/server`.

</details>

## Integration

Example integration using only the builtins of your JS Runtime:

{% content-ref url="node-http.md" %}
[node-http.md](node-http.md)
{% endcontent-ref %}

{% content-ref url="deno.serve.md" %}
[deno.serve.md](deno.serve.md)
{% endcontent-ref %}

{% content-ref url="bun.serve.md" %}
[bun.serve.md](bun.serve.md)
{% endcontent-ref %}

{% content-ref url="cloudflare-workers.md" %}
[cloudflare-workers.md](cloudflare-workers.md)
{% endcontent-ref %}

{% content-ref url="vercel-edge.md" %}
[vercel-edge.md](vercel-edge.md)
{% endcontent-ref %}

Example integration at the API framwork level

{% content-ref url="express.js.md" %}
[express.js.md](express.js.md)
{% endcontent-ref %}

{% content-ref url="fastify.md" %}
[fastify.md](fastify.md)
{% endcontent-ref %}

{% content-ref url="hono.md" %}
[hono.md](hono.md)
{% endcontent-ref %}

{% content-ref url="nest.js.md" %}
[nest.js.md](nest.js.md)
{% endcontent-ref %}

{% content-ref url="../tanstack-start.md" %}
[tanstack-start.md](../tanstack-start.md)
{% endcontent-ref %}

## TODO List Example

A TODO list example app built with Vite / React / TanStack Router on the frontend, and Node.js / Hono on the backend.

{% embed url="https://youtu.be/33VijFArY9s" %}

The app is live here:

{% embed url="https://vite-insee-starter.demo-domain.ovh/" %}

Source code (REST API):

{% embed url="https://github.com/InseeFrLab/todo-rest-api" %}

Source code (frontend):

{% embed url="https://github.com/InseeFrLab/vite-insee-starter" %}
