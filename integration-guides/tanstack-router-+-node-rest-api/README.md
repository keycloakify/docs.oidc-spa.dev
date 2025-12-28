---
description: Creating a OAuth2 enabled resource server.
icon: arrow-right-arrow-left
---

# Backend Token Validationè

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

<table data-view="cards"><thead><tr><th data-card-target data-type="content-ref">Docs</th><th data-hidden>Option</th></tr></thead><tbody><tr><td><a href="nest.js.md">nest.js.md</a></td><td>NestJS</td></tr><tr><td><a href="trpc.md">trpc.md</a></td><td>tRPC</td></tr><tr><td><a href="express.js.md">express.js.md</a></td><td>Express.js</td></tr><tr><td><a href="koa.md">koa.md</a></td><td>Koa</td></tr><tr><td><a href="fastify.md">fastify.md</a></td><td>Fastify</td></tr><tr><td><a href="hono.md">hono.md</a></td><td>Hono</td></tr><tr><td><a href="../tanstack-start.md">tanstack-start.md</a></td><td>TanStack Start</td></tr></tbody></table>

<details>

<summary>JS Runtime level integration</summary>

<table data-view="cards"><thead><tr><th>Option</th><th data-card-target data-type="content-ref">Docs</th></tr></thead><tbody><tr><td>Node.js `node:http`</td><td><a href="node-http.md">node-http.md</a></td></tr><tr><td>Deno `Deno.serve`</td><td><a href="deno.serve.md">deno.serve.md</a></td></tr><tr><td>Bun `Bun.serve`</td><td><a href="bun.serve.md">bun.serve.md</a></td></tr><tr><td>Cloudflare Workers</td><td><a href="cloudflare-workers.md">cloudflare-workers.md</a></td></tr><tr><td>Vercel Edge</td><td><a href="vercel-edge.md">vercel-edge.md</a></td></tr></tbody></table>

</details>

## Websocket

{% content-ref url="websocket.md" %}
[websocket.md](websocket.md)
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
