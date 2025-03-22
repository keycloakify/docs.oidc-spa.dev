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

This is optional but recommended for better performances and security.

{% tabs %}
{% tab title="Vite" %}
First rename your entry point file from `main.tsx` (or `main.ts`) to `main.lazy.tsx`

```bash
mv src/main.tsx src/main.lazy.tsx
```

The create a new `main.tsx` file: &#x20;

{% code title="src/main.tsx" %}
```typescript
import { oidcEarlyInit } from "oidc-spa/entrypoint";

const { shouldLoadApp } = oidcEarlyInit({
    freezeFetch: true,
    freezeXMLHttpRequest: true
});

if (shouldLoadApp) {
    import("./main.lazy");
}
```
{% endcode %}
{% endtab %}

{% tab title="React-Router Framwork Mode" %}
If you already have an `entry.client.tsx` file, rename it to `entry.client.lazy.tsx`

```bash
mv app/entry.client.tsx app/entry.client.lazy.tsx
```

If you don't, create the file `app/entry.client.tsx`: &#x20;

{% code title="app/entry.client.lazy.tsx" %}
```tsx
import React from "react";
import ReactDOM from "react-dom/client";
import { HydratedRouter } from "react-router/dom";

ReactDOM.hydrateRoot(
    document,
    <React.StrictMode>
        <HydratedRouter />
    </React.StrictMode>
);
```
{% endcode %}

Create a new app/entry.client.tsx file:

```tsx
import { oidcEarlyInit } from "oidc-spa/entrypoint";

const { shouldLoadApp } = oidcEarlyInit({
    freezeFetch: true,
    freezeXMLHttpRequest: true
});

if (shouldLoadApp) {
    import("./entry.client.lazy");
}
```
{% endtab %}

{% tab title="Create-React-App" %}
First rename your entry point file from `main.tsx` (or `main.ts`) to `main.lazy.tsx`

```bash
mv src/index.tsx src/index.lazy.tsx
```

The create a new `index.tsx` file: &#x20;

{% code title="src/index.tsx" %}
```typescript
import { oidcEarlyInit } from "oidc-spa/entrypoint";

const { shouldLoadApp } = oidcEarlyInit({
    freezeFetch: true,
    freezeXMLHttpRequest: true
});

if (shouldLoadApp) {
    import("./index.lazy");
}
```
{% endcode %}
{% endtab %}
{% endtabs %}
