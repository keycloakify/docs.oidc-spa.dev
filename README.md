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

This is optional but recommended for [better performances and security](why-oidcearlyinit.md).

{% tabs %}
{% tab title="Vite" %}
First rename your entry point file from `main.tsx` (or `main.ts`) to `main.lazy.tsx`

```bash
mv src/main.tsx src/main.lazy.tsx
```

Then create a new `main.tsx` file: &#x20;

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

{% tab title="TanStack Start" %}
Comming soon, [follow progress](https://github.com/keycloakify/oidc-spa/issues/43).
{% endtab %}

{% tab title="React-Router Framework Mode" %}
You can skip this for now. It will be explained in the dedicated setup guide:

{% content-ref url="example-setups/react-router.md" %}
[react-router.md](example-setups/react-router.md)
{% endcontent-ref %}
{% endtab %}

{% tab title="Create-React-App" %}
First rename your entry point file from `main.tsx` (or `main.ts`) to `main.lazy.tsx`

```bash
mv src/index.tsx src/index.lazy.tsx
```

Then create a new `index.tsx` file: &#x20;

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
