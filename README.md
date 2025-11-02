---
icon: sign-posts-wrench
---

# Getting Started

{% hint style="info" %}
If you're having issues do not hesitate to [reach out on Discord](https://discord.gg/mJdYJSdcm4) we're here to help!
{% endhint %}

## What this is

oidc-spa is a framework-agnostic OpenID Connect client for browser-centric web applications implementing the [Authorization Code Flow with PKCE](resources/why-no-client-secret.md). &#x20;

It replaces provider-specific SDKs like [keycloak-js](https://www.npmjs.com/package/keycloak-js), [auth0-spa-js](https://www.npmjs.com/package/@auth0/auth0-spa-js), or [@azure/msal-browser](https://www.npmjs.com/package/@azure/msal-browser) with one unified API that works with Keycloak, Auth0, Entra ID, and any other spec-compliant OIDC provider.

oidc-spa provides strong guarantees regarding the [protection of your tokens **even in case of successful XSS or supply chain attacks**](resources/why-no-client-secret.md#how-oidc-spa-mitigates-the-risks-of-token-exposure). No other solution does that.

oidc-spa is uncompromising in terms of performance, security, DX, and UX. You get a state-of-the-art authentication and authorization system out of the box with zero glue code to write and no knobs to adjust.

Unlike server-centric solutions such as [NextAuth](https://next-auth.js.org/), oidc-spa makes the frontend the OIDC client.

Your backend becomes a simple OAuth2 resource server, and tokens can be validated offline. oidc-spa [also provides the tools for token validation on the server side](integration-guides/tanstack-router-+-node-rest-api.md).

That means no database, no session store, and **enterprise-grade UX** out of the box, while scaling naturally to edge runtimes.

oidc-spa exposes real OIDC primitives, decoded ID tokens, access tokens, and claims, instead of hiding them behind a “user” object, helping you understand and control your security posture.

It’s infra-light, open-standard, transparent, and ready to work in minutes.

<details>

<summary>But in details? I want to understand the tradeoffs.</summary>

In the modern tech ecosystem, no one “rolls their own auth” anymore, not even OpenAI or Vercel.\
Authentication has become a **platform concern**. Whether you host your own identity provider like **Keycloak**, or use a service such as **Auth0** or **Microsoft Entra ID**, authentication today means **redirecting users to your auth provider**.

***

**The problem oidc-spa solves**

Each provider ships its own bespoke SDK, `keycloak-js`, `auth0-spa-js`, `MSAL.js`, etc.\
So even though they all implement the same open standard, **you end up vendor-locking your application** and restricting where it can be deployed.

We need a solution that allows you to talk to _any_ provider through a **unified, open, provider-agnostic API**.\
That’s what oidc-spa brings to the table.

***

**Why not NextAuth or similar?**

Some “agnostic” solutions like **NextAuth** exist, but they’re **backend-centric**, in the OpenID Connect model, the _server_ is the OIDC client.\
This makes authentication **tightly coupled to a specific stack** (e.g. Next.js for NextAuth) and **requires maintaining a database**.

These server-side solutions also **hide too much**. You’re left with a “user” object and no clear understanding of what your actual security posture is.

As for solution like oidc-client-ts or react-oidc-context. They are good for what they are but requires month of integration for acheiving what oidc-spa gives you out of the box.\
oidc-spa is internally using a vendored version of oidc-client-ts.

***

**The oidc-spa model**

With **oidc-spa**, the **frontend is the OIDC client**.\
The backend becomes a simple **resource server**, which you call using **access tokens** that can be **validated offline**, oidc-spa provides the tools for that too.

This model is **extremely light on infrastructure** and trivial to set up.\
You don’t need a database. You just provide your IdP credentials and instantly get **enterprise-grade UX** with **zero integration code**.

It also **scales infinitely**: authentication load is decentralized, since each client communicates directly with the auth server.\
For edge runtimes, this yields a real **performance advantage**, no need to restore a session from a central store before doing work.

And unlike abstract “user” APIs, oidc-spa works directly with **ID tokens, access tokens, and claims**, the real building blocks of authentication and authorization on the modern web.\
You’re learning _transferable knowledge_ that can make you CTO material, not just another black-box SDK.

***

**Why this isn’t already standard**

There’s little financial incentive to build such a solution.\
Every IdP has a vested interest in **locking you in**, so you’ll hit the limits of the free tier and start paying.\
oidc-spa is different, it’s a **tool, not a platform**. We devlop it because we need it and for the love of the game.

***

**About SSR and modern rendering**

When the **browser owns the auth**, traditional **server-side rendering** becomes trickier.\
That’s why oidc-spa primarily targets **single-page applications**.

That said, there’s a **full-stack story** through **TanStack Start**, which provides primitives to SSR as deeply as possible, then defer user-specific rendering to the client.\
So while oidc-spa doesn’t support _full-page_ SSR, no serious app really does, not even Clerk or Vercel.\
They all render a shell first, then progressively stream authenticated content. oidc-spa achieves the **same UX**, with **equal or better performance**.

***

**Security and XSS resilience**

Yes; client-side authentication raises valid security concerns.\
But this isn’t a fatal flaw; it’s an **engineering challenge**, and oidc-spa addresses it head-on.

It treats the browser as a **hostile environment**, going to great lengths to protect tokens even under **XSS or supply-chain attacks**.\
These mitigations [are documented here](resources/why-no-client-secret.md).

***

**Known tradeoffs**

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

> TanStack Start has a special status since your getting both the frontend and backend capabilities of oidc-spa integrated in a single adapter!

{% content-ref url="integration-guides/tanstack-start.md" %}
[tanstack-start.md](integration-guides/tanstack-start.md)
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
