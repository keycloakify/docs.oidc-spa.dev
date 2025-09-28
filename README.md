---
icon: sign-posts-wrench
---

# Installation

{% hint style="info" %}
Before starting be aware that oidc-spa is not suited for Next.js.

If you are using Next the closer alternative is to use [NextAuth.js](https://next-auth.js.org/) (with [the Keycloak adapter](https://next-auth.js.org/providers/keycloak) if you are using Keycloak). See [this guide](https://phasetwo.io/docs/securing-applications/next/).
{% endhint %}

If you're having issues don't hesitate to [reach out on Discord](https://discord.gg/mJdYJSdcm4)!

## Add the lib to your dependencies

{% tabs %}
{% tab title="npm" %}
```bash
npm install oidc-spa
```
{% endtab %}

{% tab title="yarn" %}
```bash
yarn add oidc-spa
```
{% endtab %}

{% tab title="pnpm" %}
```bash
pnpm add oidc-spa
```
{% endtab %}

{% tab title="bun" %}
```bash
bun add oidc-spa
```
{% endtab %}
{% endtabs %}

## Editing your App entrypoint

To protect tokens against supply-chain attacks and XSS, oidc-spa must run some initialization code _before any other JavaScript in your app_.

This design provides much stronger security guarantees than any other adapter, and it also delivers unmatched login performance. More details [here](why-oidcearlyinit.md).

{% tabs %}
{% tab title="Vite" %}
First rename your entry point file from `main.tsx` (or `main.ts`) to `main.lazy.tsx`

```bash
mv src/main.tsx src/main.lazy.tsx
```

Then create a new `main.tsx` file:

{% code title="src/main.tsx" %}
```typescript
import { oidcEarlyInit } from "oidc-spa/entrypoint";

const { shouldLoadApp } = oidcEarlyInit({
    freezeFetch: true,
    freezeXMLHttpRequest: true,
    freezeWebSocket: true
});

if (shouldLoadApp) {
    // Note: Deferring the main app import adds a few milliseconds to cold start,
    // but dramatically speeds up auth. Overall, it's a net win.
    import("./main.lazy");
}
```
{% endcode %}
{% endtab %}

{% tab title="TanStack Start" %}
Comming soon, [follow progress](https://github.com/keycloakify/oidc-spa/issues/43).
{% endtab %}

{% tab title="React-Router Framework" %}
You can skip this for now. It will be explained in the dedicated setup guide:

{% content-ref url="setup-guides/react-router.md" %}
[react-router.md](setup-guides/react-router.md)
{% endcontent-ref %}
{% endtab %}

{% tab title="Angular" %}
> WARNING: The Angular adapter is still subject to changes!

First rename your entry point file from `main.ts` to `main.lazy.ts`

```bash
mv src/main.tsx src/main.lazy.tsx
```

Then create a new `main.ts` file:

{% code title="main.ts" %}
```typescript
import { oidcEarlyInit } from "oidc-spa/entrypoint";

const { shouldLoadApp } = oidcEarlyInit({
    freezeFetch: true,
    freezeXMLHttpRequest: true,
    freezeWebSocket: true
});

if (shouldLoadApp) {
    // Note: Deferring the main app import adds a few milliseconds to cold start,
    // but dramatically speeds up auth. Overall, it's a net win.
    import("./main.lazy");
}
```
{% endcode %}
{% endtab %}

{% tab title="CRA" %}
{% hint style="warning" %}
Create React App is deprecated. Consider using Vite instead.
{% endhint %}

First rename your entry point file from `main.tsx` (or `main.ts`) to `main.lazy.tsx`

```bash
mv src/index.tsx src/index.lazy.tsx
```

Then create a new `index.tsx` file:

{% code title="src/index.tsx" %}
```typescript
import { oidcEarlyInit } from "oidc-spa/entrypoint";

const { shouldLoadApp } = oidcEarlyInit({
    freezeFetch: true,
    freezeXMLHttpRequest: true,
    freezeWebSocket: true
});

if (shouldLoadApp) {
    // Note: Deferring the main app import adds a few milliseconds to cold start,
    // but dramatically speeds up auth. Overall, it's a net win.
    import("./index.lazy");
}
```
{% endcode %}
{% endtab %}
{% endtabs %}
