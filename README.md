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

# Getting Started

{% hint style="info" %}
If you're having issues, don't hesitate to [reach out on Discord](https://discord.gg/mJdYJSdcm4). We're here to help!
{% endhint %}

## What this is

oidc-spa is an OpenID Connect client for browser-centric web apps. It implements the [Authorization Code Flow with PKCE+DPoP](resources/why-no-client-secret.md)  and also provides [token validation utilities for JavaScript backends](integration-guides/backend-token-validation/). &#x20;

It features [a set of security defences](security-features/overview.md) that makes it stand out compared to other Client-Side OIDC implementation.

It’s a single library that can replace platform-specific SDKs like keycloak-js, MSAL.js, @auth0/auth0-spa-js, etc. on the frontend, and [jsonwebtoken](https://www.npmjs.com/package/jsonwebtoken), [jose](https://www.npmjs.com/package/jose) or [express-jwt](https://www.npmjs.com/package/express-jwt) on your JS backend.

Here is an comparison to help you understand better where oidc-spa sits:

<table><thead><tr><th width="172.53125"></th><th>Client-Side OIDC</th><th>Server-Side OIDC</th></tr></thead><tbody><tr><td><strong>Implementation</strong></td><td><strong>oidc-spa</strong>, keycloak-js, angular-oauth2-oidc, react-oidc-context, @auth0/auth0-spa-js, @azure/msal-browser, @axa-fr/oidc-client, oidc-client-ts (without client secret)</td><td>nuxt-oidc-auth, oidc-client-ts (with client secret), NextAuth/Auth.js/BetterAuth (Ish they are "roll your own auth" solutions that can funnel OIDC providers)</td></tr><tr><td><strong>OIDC Model</strong></td><td>The fronted code is the OIDC client. Your backend API is a OAuth resource server. The frontend makes request with the access token in the header to the API. The API can resolve the identity offline by validating the signature of the token.</td><td>The backend is the OIDC client. The user identity of the user is tracked across request with session cookies. In this model there is usually no notion of OAuth resource server. The access token is not used uless you call third pary services.</td></tr><tr><td><strong>Infra requirement</strong></td><td><mark style="color:$success;">None. The browser talks directly to the Auth Server.</mark></td><td><mark style="color:$warning;">Requires a statefull backend and a store (eg redis) to share session across server replicate.</mark></td></tr><tr><td><strong>Set up simplicity</strong></td><td><mark style="color:$success;">Very easy. Auth concerns are decoupled from your app framework, routing library and backend API.</mark></td><td><mark style="color:$warning;">Strongly coupled with a specific framwork, need to create login/logout routes and setup middlewares.</mark></td></tr><tr><td><strong>Security</strong></td><td><mark style="color:$warning;">Historyically much weaker, tokens are exposed to the frontend code.</mark><br><mark style="color:$success;">Today, with DPoP</mark> <a href="security-features/overview.md"><mark style="color:$success;">and other measures enabled</mark></a><mark style="color:$success;">, security is just as strong.</mark></td><td><mark style="color:$success;">Secure by design. The tokens are never exposed to the fronted.</mark></td></tr><tr><td><strong>Server Side Rendering</strong></td><td><mark style="color:$warning;">Limited. The server do not know who the user is when rendering the pages. Only public pages and global layout can be SSR'd. Auth aware component rendering must be delegated to the client.</mark></td><td><mark style="color:$success;">Seamless. The server know who the user is.</mark></td></tr></tbody></table>

### Is it a good fit for my stack? &#x20;

It depends, while oidc-spa clearly surpasses other solutions in the Client-Side OIDC category, client-Side OIDC is not the right model for all applications.

#### When NOT to use oidc-spa

If you use a full stack JS framwork with SSR enabled: Next.js, Nuxt, SvelteKit, Remix/React Router Framwork (not in SPA mode) or Astro. oidc-spa is probably not a good choice. &#x20;

Those framwork end goal is to move as much of the states and logic to the backend and send as little JavaScript to the client as possible. &#x20;

In oidc-spa the auth is driven by the frontend, it's optimized for highly interactive web application, not for content driven websites. There is a philosophy missmatch here. &#x20;

#### When you should use it

Basically any project where there is no Server Side Rendering or where SSR is just used primarely for SEO and to improve TFP but where the core of the logic and states lives in the browser.

Typically: &#x20;

* Vite + React (or another UI framework)
* TanStack Start
* Angular applications
* Nuxt with SSR: false.
* React Router Framwork with SSR false.

If you're hessitating between this and implementing a BFF pattern for security considerations you can checkout [the security features of oidc-spa](security-features/overview.md). With those defence enabled, the security profile of applications matches those implementing Server Side OIDC.

***

## Getting Started

Onboard? Pick the right integration path for your stack.

{% content-ref url="integration-guides/tanstack-router-start/" %}
[tanstack-router-start](integration-guides/tanstack-router-start/)
{% endcontent-ref %}

{% content-ref url="integration-guides/react-router.md" %}
[react-router.md](integration-guides/react-router.md)
{% endcontent-ref %}

{% content-ref url="integration-guides/angular.md" %}
[angular.md](integration-guides/angular.md)
{% endcontent-ref %}

{% content-ref url="integration-guides/usage.md" %}
[usage.md](integration-guides/usage.md)
{% endcontent-ref %}

***
