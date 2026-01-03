---
icon: sign-posts-wrench
layout:
  width: default
  title:
    visible: true
  description:
    visible: false
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: false
  metadata:
    visible: true
metaLinks:
  alternates:
    - https://app.gitbook.com/s/UhNOMoIddws1XoAnT5Nn/
---

# What This Is

{% hint style="info" %}
Stuck? Reach out on [Discord](https://discord.gg/mJdYJSdcm4). We’ll help you debug it.
{% endhint %}

oidc-spa is an OpenID Connect client for browser-first web apps. It implements the [Authorization Code Flow with PKCE](resources/why-no-client-secret.md) and supports [DPoP](security-features/dpop.md). It also ships [token validation utilities for JavaScript backends](integration-guides/backend-token-validation/).

It includes [security defenses](security-features/overview.md) to reduce token exposure risks in the browser.

It’s one library for the full stack. It can replace frontend SDKs like `keycloak-js`, `MSAL.js`, or `@auth0/auth0-spa-js`. It can also replace backend token tooling like `jsonwebtoken`, `jose`, or `express-jwt`.

**What it isn't**

* It is **not** a server-side auth/session framework.
* If auth state must exist on the server at render time, this isn't your tool.

## Dive In

Ready to integrate? Start here.

{% content-ref url="integration-guides/example-setups.md" %}
[example-setups.md](integration-guides/example-setups.md)
{% endcontent-ref %}

## Positioning

Here’s where oidc-spa sits compared to server-side OIDC:

<table><thead><tr><th width="172.53125"></th><th>Client-Side OIDC</th><th>Server-Side OIDC</th></tr></thead><tbody><tr><td><strong>Implementation</strong></td><td><strong><code>oidc-spa</code></strong>, <code>keycloak-js</code>, <code>angular-oauth2-oidc</code>, <code>react-oidc-context</code>, <code>@auth0/auth0-spa-js</code>, <code>@azure/msal-browser</code>, <code>@axa-fr/oidc-client</code>, <code>oidc-client-ts</code> (without client secret)</td><td><a href="https://nuxtoidc.cloud/"><code>nuxt-oidc-auth</code></a>, <code>oidc-client-ts</code> (with client secret), <code>NextAuth</code>/<code>Auth.js</code>/<code>BetterAuth</code> (often “roll your own auth” frameworks that can broker OIDC providers)</td></tr><tr><td><strong>OIDC Model</strong></td><td>The frontend is the OIDC client. Your backend API is an OAuth resource server. The frontend calls the API with an access token. The API can validate the token signature and resolve identity offline.</td><td>The backend is the OIDC client. User identity is tracked with session cookies. In this model, there is usually no OAuth resource server. Access tokens are mainly used for calling third-party APIs.</td></tr><tr><td><strong>Infrastructure</strong></td><td><mark style="color:$success;">None. The browser talks directly to the authorization server.</mark></td><td><mark style="color:$warning;">Requires a stateful backend and a shared session store (e.g. Redis).</mark></td></tr><tr><td><strong>Setup</strong></td><td><mark style="color:$success;">Usually simple. Auth is decoupled from your app framework, router, and API.</mark></td><td><mark style="color:$warning;">Tightly coupled to a framework. You typically build login/logout routes and middleware.</mark></td></tr><tr><td><strong>Security</strong></td><td><mark style="color:$warning;">Historically weaker because tokens exist in the browser.</mark><br><mark style="color:$success;">With DPoP and modern defenses, the security gap can shrink significantly.</mark> <a href="security-features/overview.md"><mark style="color:$success;">See details</mark></a><mark style="color:$success;">.</mark></td><td><mark style="color:$success;">Secure by design. Tokens are not exposed to frontend code.</mark></td></tr><tr><td><strong>Server-side rendering</strong></td><td><mark style="color:$warning;">Limited. The server renders without user context. Auth-aware UI renders on the client.</mark></td><td><mark style="color:$success;">Seamless. The server knows who the user is during render.</mark></td></tr></tbody></table>

## Is it a good fit for my stack?

It depends. oidc-spa is strong for client-side OIDC. But client-side OIDC isn’t the right model for every app.

### When NOT to use oidc-spa

Avoid oidc-spa if you rely on SSR for auth-aware pages. This includes Next.js, Nuxt, SvelteKit, Remix / React Router Framework (non‑SPA mode), or Astro.

Those stacks push state and logic to the server. They also aim to ship minimal client JavaScript.

oidc-spa drives auth from the browser. That’s a mismatch for SSR-first architectures.

### When you should use it

Use it for client-first apps. It works best when state and logic live in the browser.

Typically:

* Vite + React (or another UI framework) - SPAs
* TanStack Start (SSR works, but auth-aware UI renders client-side)
* Angular applications
* Nuxt with `ssr: false`
* React Router Framework with `ssr: false`

If you’re choosing between this and a BFF for security, start with the [security features](security-features/overview.md). With those defenses enabled, the security profile can be comparable to server-side OIDC.

***

Next step: get your app authenticated.

{% content-ref url="integration-guides/example-setups.md" %}
[example-setups.md](integration-guides/example-setups.md)
{% endcontent-ref %}
