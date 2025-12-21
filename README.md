---
icon: sign-posts-wrench
---

# Getting Started

{% hint style="info" %}
If you're having issues do not hesitate to [reach out on Discord](https://discord.gg/mJdYJSdcm4) we're here to help!
{% endhint %}

## What this is

oidc-spa is a framework-agnostic OpenID Connect client for browser-centric web applications, implementing the [Authorization Code Flow with PKCE](resources/why-no-client-secret.md), and optionally providing OAuth JWT access token validation utilities for JavaScript backends.  \
\
It’s a single authentication library you can use to integrate with Keycloak, Microsoft Entra ID, Auth0, Clerk, etc.

<details>

<summary>But in details?</summary>

It work with any spec compliant OIDC provider like [Keycloak](https://www.keycloak.org/), [Auth0](https://auth0.com/) or [Microsoft EntraID](https://www.microsoft.com/fr-fr/security/business/identity-access/microsoft-entra-id) and replace provider-specific SDKs like [keycloak-js](https://www.npmjs.com/package/keycloak-js), [auth0-spa-js](https://www.npmjs.com/package/@auth0/auth0-spa-js), or [@azure/msal-browser](https://www.npmjs.com/package/@azure/msal-browser) with one unified API, freeing your app from vendor lock-in and making it deployable in any IT system. Concretly this mean that it let you build an app and sell it to different companies ensuring they will be able to deploy it in their environement regardless of what auth plafrom they use internally.

oidc-spa provides strong guarantees regarding the protection of your tokens [**even in case of successful XSS or supply chain attacks**](https://docs.oidc-spa.dev/resources/xss-and-supply-chain-attack-protection). No other implementation can currently claim that.

It is uncompromising in terms of performance, security, DX, and UX. You get a state-of-the-art authentication and authorization system out of the box with zero glue code to write and no knobs to adjust.

Unlike server-centric solutions such as [Auth.js](https://authjs.dev/), oidc-spa makes the frontend the OIDC client in your IdP model's representation.

Your backend becomes a simple OAuth2 resource server that you frontend query with the access token attached as Authorization header. oidc-spa also provides the tools for token validation on the server side:

* As an unified solution for [TanStack Start](https://tanstack.com/router/latest)
* Or, as [a separate adapter](integration-guides/tanstack-router-+-node-rest-api.md) for creating authed APIs with [tRCP](https://trpc.io/), [Express](https://expressjs.com/), [Hono](https://hono.dev/), [Nest.js](https://nestjs.com/), ect.

That means **no database,** no session store, and **enterprise-grade UX** out of the box, while scaling naturally to edge runtimes.

oidc-spa exposes real OIDC primitives, decoded ID tokens, access tokens, and claims, instead of hiding them behind a “user” object, helping you understand and control your security posture.

It’s infra-light, open-standard, transparent, and ready to work in minutes.

***

In the modern tech ecosystem, no one “rolls their own auth” anymore, not even OpenAI or Vercel.\
Authentication has become a **platform concern**. Whether you host your own identity provider like **Keycloak**, or use a service such as **Auth0** or **Microsoft Entra ID**, authentication today means **redirecting users to your auth provider**.

***

**Why not** [**BetterAuth**](https://www.better-auth.com/) **or** [**Auth.js**](https://authjs.dev/)

These are great for what they are, but they’re “roll your own auth” solutions.\
With oidc-spa, you delegate authentication to a specialized identity provider such as Keycloak, Auth0, Okta, or Clerk.

With BetterAuth, your backend _is_ the authorization server, even if you can integrate third party identity providers id doesn't change that fact.\
That’s very battery-included, but also far heavier infrastructure-wise.\\

Another big difference: oidc-spa is **browser-centric**. The token exchange happens on the client,\
and the backend server is merely an OAuth2 resource server in the OIDC model.

If you use BetterAuth to provide login via Keycloak, your backend becomes the OIDC client application,\
which has some security benefits over browser token exchange, but at the cost of centralization and requiring backend infrastructure.

One clear advantage BetterAuth has over oidc-spa is more natural SSR support. In the oidc-spa model, the server doesn’t know the authentication state of the user at all time, which makes it difficult to integrate with traditional full-stack frameworks that rely on server-side rendering.

***

**Server Side Rendering**

The only SSR-capable framework we currently support is [TanStack Start](https://tanstack.com/start/latest), because it provides the low-level primitives needed to render as much as possible on the server while deferring rendering of auth aware components to the client.

This approach achieves a similar UX and performance to server-centric frameworks, but it’s inherently less transparent than streaming fully authenticated components to the client.

Try the TansStack Start example deployment with JavaScript disabled to get a feel of what can and can't be SSR'd: [https://example-tanstack-start.oidc-spa.dev/](https://example-tanstack-start.oidc-spa.dev/)

***

**Security and XSS resilience**

Yes; client-side authentication raises valid security concerns.\
But this isn’t a fatal flaw; it’s an **engineering challenge**, and oidc-spa addresses it head-on.

It treats the browser as a **hostile environment**, going to great lengths to protect tokens even under **XSS or supply-chain attacks**.\
These mitigations [are documented here](features/token-exfiltration-defence.md).

***

**Limitations regarding backend delegation**

The main limitation is with **long-running background operations**.\
If your backend must call third-party APIs **on behalf of the user** while they’re offline, you’ll need **service accounts** for those APIs or take charge of rotating tokens yourself which [can be tricky](https://authjs.dev/guides/refresh-token-rotation).\
Beyond that, everything else scalability, DX, performance, works in your favor.

***

If that all sounds good to you…\
**Let’s get started.**

</details>

***

## Configuring your IdP

You can skip this for now since all our examples comes with demo Keycloak/Auth0/EntraID/Google accounts that you can freely use for development.\
Eventually however you'll need to configure your own account/instance.

{% content-ref url="providers-configuration/provider-configuration.md" %}
[provider-configuration.md](providers-configuration/provider-configuration.md)
{% endcontent-ref %}

***

## Integration

At its core, oidc-spa is a **framework-agnostic solution for client-centric web applications**. It’s not tied to any specific UI framwork.

In an effort to minimize the amount of glue code you have to write we also provide framework-specific adapters for popular environments.

Pick one:

{% content-ref url="integration-guides/tanstack-router-start/" %}
[tanstack-router-start](integration-guides/tanstack-router-start/)
{% endcontent-ref %}

{% content-ref url="setup-guides/react-router.md" %}
[react-router.md](setup-guides/react-router.md)
{% endcontent-ref %}

{% content-ref url="integration-guides/angular.md" %}
[angular.md](integration-guides/angular.md)
{% endcontent-ref %}

{% content-ref url="integration-guides/usage.md" %}
[usage.md](integration-guides/usage.md)
{% endcontent-ref %}

***
