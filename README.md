---
icon: sign-posts-wrench
---

# Getting Started

{% hint style="info" %}
If you're having issues do not hesitate to [reach out on Discord](https://discord.gg/mJdYJSdcm4) we're here to help!
{% endhint %}

## What this is

oidc-spa is a framework-agnostic OpenID Connect client for browser-centric web applications, implementing the [Authorization Code Flow with PKCE](resources/why-no-client-secret.md), and also a token validation solution for JavaScript backends.  \
It's a single library that you can use to integrate with Keycloak, Microsoft Entra ID, Auth0, Clerk and any OIDC compliant provider and that can be used as a replacement to platform specific SDK like keycloak-js, MSAL.js, @auth0/auth0-spa-js ect.

**Is it a good fit for my stack?**&#x20;

oidc-spa shines in apps where the logic and states live primarely in the browser, so any Single Page Application, or frontend oriented framwork like TanStack Start.

It is not a good fit however for Next.js, Nuxt or Astro, meta framworks that tries to avoid involving the client as little as possible. In oidc-spa, the auth is drove by the browser so there is a philosophy missmatch here. &#x20;

<details>

<summary>More context</summary>

In the modern tech ecosystem, no one “rolls their own auth” anymore, not even OpenAI or Vercel.\
Authentication has become a **platform concern**. Whether you host your own identity provider like **Keycloak**, or use a service such as **Auth0** or **Microsoft Entra ID**, authentication today means **redirecting users to your auth provider**.

***

**What's the core diffrence with** [**BetterAuth**](https://www.better-auth.com/) **or** [**Auth.js**](https://authjs.dev/)

These are “roll your own auth” solutions.\
With oidc-spa, you delegate authentication to a specialized identity provider such as Keycloak, Auth0, Okta, or Clerk.

With BetterAuth or Auth.js, your backend _is_ the authorization server, even if you can integrate third party identity providers id doesn't change that fact.\
That’s very battery-included, but also far heavier infrastructure-wise.

Another big difference: oidc-spa is **browser-centric**. The token exchange happens on the client, the backend server is merely an OAuth2 resource server in the OIDC model.

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
These mitigations [are documented here](resources/token-exfiltration-defence.md).

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
Eventually however you'll need to configure your own credentials.

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
