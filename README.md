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
If you're having issues, don't hesitate to [reach out on Discord](https://discord.gg/mJdYJSdcm4). We're here to help!
{% endhint %}

oidc-spa is an OpenID Connect client for browser-centric web apps. It implements the [Authorization Code Flow with PKCE](resources/why-no-client-secret.md)+[DPoP](security-features/dpop.md) and also provides [token validation utilities for JavaScript backends](integration-guides/backend-token-validation/).

It features [a set of security defences](security-features/overview.md) that make it stand out compared to other Client-Side OIDC implementations.

It’s a single library that can replace platform-specific SDKs like keycloak-js, MSAL.js, @auth0/auth0-spa-js, etc. on the frontend, and [jsonwebtoken](https://www.npmjs.com/package/jsonwebtoken), [jose](https://www.npmjs.com/package/jose) or [express-jwt](https://www.npmjs.com/package/express-jwt) on your JS backend.

## Dive In

Convinced this is what you're looking for already? Let's get your app authenticated.

{% content-ref url="integration-guides/example-setups.md" %}
[example-setups.md](integration-guides/example-setups.md)
{% endcontent-ref %}

## Positioning

Here is a comparison to help you understand better where oidc-spa sits:

<table><thead><tr><th width="172.53125"></th><th>Client-Side OIDC</th><th>Server-Side OIDC</th></tr></thead><tbody><tr><td><strong>Implementation</strong></td><td><strong>oidc-spa</strong>, keycloak-js, angular-oauth2-oidc, react-oidc-context, @auth0/auth0-spa-js, @azure/msal-browser, @axa-fr/oidc-client, oidc-client-ts (without client secret)</td><td>nuxt-oidc-auth, oidc-client-ts (with client secret), NextAuth/Auth.js/BetterAuth (Ish they are "roll your own auth" solutions that can funnel OIDC providers)</td></tr><tr><td><strong>OIDC Model</strong></td><td>The frontend code is the OIDC client. Your backend API is an OAuth resource server. The frontend makes requests with the access token in the header to the API. The API can resolve the identity offline by validating the signature of the token.</td><td>The backend is the OIDC client. The user identity of the user is tracked across requests with session cookies. In this model there is usually no notion of an OAuth resource server. The access token is not used unless you call third-party services.</td></tr><tr><td><strong>Infra requirement</strong></td><td><mark style="color:$success;">None. The browser talks directly to the Auth Server.</mark></td><td><mark style="color:$warning;">Requires a stateful backend and a store (eg redis) to share session across server replicas.</mark></td></tr><tr><td><strong>Set up simplicity</strong></td><td><mark style="color:$success;">Very easy. Auth concerns are decoupled from your app framework, routing library and backend API.</mark></td><td><mark style="color:$warning;">Strongly coupled with a specific framework, need to create login/logout routes and set up middlewares.</mark></td></tr><tr><td><strong>Security</strong></td><td><mark style="color:$warning;">Historically much weaker, tokens are exposed to the frontend code.</mark><br><mark style="color:$success;">Today, with DPoP and other modern defences, there is a case to be made that both security profiles are equivalent.</mark> <a href="security-features/overview.md"><mark style="color:$success;">Discussed here</mark></a><mark style="color:$success;">.</mark></td><td><mark style="color:$success;">Secure by design. The tokens are never exposed to the frontend.</mark></td></tr><tr><td><strong>Server Side Rendering</strong></td><td><mark style="color:$warning;">Limited. The server does not know who the user is when rendering the pages. Only public pages and global layout can be SSR'd. Auth aware component rendering must be delegated to the client.</mark></td><td><mark style="color:$success;">Seamless. The server knows who the user is.</mark></td></tr></tbody></table>

## Is it a good fit for my stack?

It depends, while oidc-spa clearly stands out compared to other solutions in the Client-Side OIDC category, client-side OIDC is not the right model for all applications.

### When NOT to use oidc-spa

If you use a full stack JS framework with SSR enabled: Next.js, Nuxt, SvelteKit, Remix/React Router Framework (not in SPA mode) or Astro. oidc-spa is probably not a good choice.

Those frameworks' end goal is to move as much of the states and logic to the backend and send as little JavaScript to the client as possible.

In oidc-spa the auth is driven by the frontend. There is a philosophy mismatch here.

### When you should use it

Basically any client-first web application. Highly interactive applications where states and logic primarily live in the frontend.

Typically:

* Vite + React (or another UI framework) - Single Page Applications (SPAs)
* TanStack Start - SSR support but rendering of auth-aware components happens client-side.
* Angular applications
* Nuxt with SSR: false
* React Router Framework with SSR false.

If you're hesitating between this and implementing a BFF pattern for security considerations you can check out [the security features of oidc-spa](security-features/overview.md). With those defences enabled, the security profile of applications matches those implementing server-side OIDC.

***

Onboard? Let's get your app authenticated!

{% content-ref url="integration-guides/example-setups.md" %}
[example-setups.md](integration-guides/example-setups.md)
{% endcontent-ref %}
