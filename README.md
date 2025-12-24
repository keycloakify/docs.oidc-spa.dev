---
icon: sign-posts-wrench
---

# Getting Started

{% hint style="info" %}
If you're having issues, don't hesitate to [reach out on Discord](https://discord.gg/mJdYJSdcm4). We're here to help!
{% endhint %}

## What this is

oidc-spa is a framework-agnostic OpenID Connect client for browser-centric web apps. It implements the [Authorization Code Flow with PKCE](resources/why-no-client-secret.md) and also provides token validation utilities for JavaScript backends.\
It’s a single library that can replace platform-specific SDKs like `keycloak-js`, `MSAL.js`, `@auth0/auth0-spa-js`, etc.

**Is it a good fit for my stack?**

oidc-spa shines in apps where logic and state live primarily in the browser. Think single-page applications (SPAs) and frontend-oriented frameworks like [TanStack Start](https://tanstack.com/start/latest).

It’s not a good fit for [Next.js,](https://nextjs.org/) [Nuxt](https://nuxt.com/), or [Astro](https://astro.build/). These meta-frameworks try to involve the client as little as possible. In oidc-spa, auth is driven by the browser, so there’s a philosophy mismatch.

<details>

<summary>More context</summary>

In the modern tech ecosystem, no one “rolls their own auth” anymore, not even OpenAI or Vercel.\
Authentication has become a **platform concern**. Whether you host your own identity provider like **Keycloak**, or use a service such as **Auth0** or **Microsoft Entra ID**, authentication today means **redirecting users to your auth provider**.

***

**What's the core difference with** [**BetterAuth**](https://www.better-auth.com/) **or** [**Auth.js**](https://authjs.dev/)?

These are “roll your own auth” solutions.\
With oidc-spa, you delegate authentication to a specialized identity provider such as Keycloak, Auth0, Okta, or Clerk.

With BetterAuth or Auth.js, your backend _is_ the authorization server. Even if you integrate third-party identity providers, it doesn’t change that fact.\
That’s very batteries-included, but also much heavier infrastructure-wise.

Another big difference: With oidc-spa the token exchange happens on the client, and the backend server is merely an OAuth 2.0 resource server in the OIDC model. The frontend is the client application in the OIDC model.

With BetterAuth and Auth.js, the backend that renders the pages is the client application, it's the backend that exchanges tokens with the auth server. &#x20;

***

**Server Side Rendering**

The only SSR-capable framework we currently support is [TanStack Start](https://tanstack.com/start/latest), because it provides the low-level primitives needed to render as much as possible on the server while deferring rendering of auth-aware components to the client.

This approach achieves a similar UX and performance to server-centric frameworks, but it’s inherently less transparent than streaming fully authenticated components to the client.

Try the TanStack Start example deployment with JavaScript disabled to get a feel for what can and can't be SSR’d: [https://example-tanstack-start.oidc-spa.dev/](https://example-tanstack-start.oidc-spa.dev/)

***

**Security and XSS resilience**

Yes; client-side authentication raises valid security concerns.\
But this isn’t a fatal flaw; it’s an **engineering challenge**, and oidc-spa addresses it head-on.

It treats the browser as a **hostile environment**, going to great lengths to protect tokens even under **XSS or supply-chain attacks**.\
These mitigations [are documented here](resources/token-exfiltration-defence.md).

***

**Limitations regarding backend delegation**

The main limitation is with **long-running background operations**.\
If your backend must call third-party APIs **on behalf of the user** while they’re offline, you’ll need **service accounts** for those APIs or take charge of rotating tokens yourself which [can be tricky](https://authjs.dev/guides/refresh-token-rotation).\
Beyond that, everything else (scalability, DX, performance) works in your favor.

***

If that all sounds good to you…\
**Let’s get started.**

</details>

***

## Configuring your IdP

You can skip this for now. All our examples come with demo Keycloak/Auth0/Entra ID/Google accounts that you can freely use for development.\
Eventually, you’ll want to configure your own credentials.

{% content-ref url="providers-configuration/provider-configuration.md" %}
[provider-configuration.md](providers-configuration/provider-configuration.md)
{% endcontent-ref %}

***

## Integration

Pick the integration path for your stack.

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
