---
description: Use oidc-spa in a pure Nuxt SPA app.
icon: mountains
---

# Nuxt

## Before you start

{% hint style="warning" %}
This setup is SPA-only.

You must set `ssr: false` in `nuxt.config.ts`.

Authentication happens in the browser.

That means Nuxt cannot know who the user is at render time.

You give up SSR auth-at-render-time benefits and effectively run a SPA inside Nuxt.

Do not use this setup if you need SSR or hybrid auth to work as-is.
{% endhint %}

## The example

This example shows a practical Nuxt setup.

It uses the built-in `oidc-spa/nuxt-spa` support, a client plugin, a composable, and route middleware.

```bash
npx gitpick keycloakify/oidc-spa/tree/main/examples/nuxt-spa oidc-spa-nuxt-spa
cd oidc-spa-nuxt-spa
cp .env.local.sample .env.local

# Set your provider values in .env.local or enable mock mode

yarn install
yarn dev

# Start exploring with: app/plugins/01.oidc.client.ts
```

{% embed url="https://github.com/keycloakify/oidc-spa/tree/main/examples/nuxt-spa" %}

## Installation

{% tabs %}
{% tab title="npm" %}
```bash
npm install oidc-spa zod
```
{% endtab %}

{% tab title="yarn" %}
```bash
yarn add oidc-spa zod
```
{% endtab %}

{% tab title="pnpm" %}
```bash
pnpm add oidc-spa zod
```
{% endtab %}

{% tab title="bun" %}
```bash
bun add oidc-spa zod
```
{% endtab %}
{% endtabs %}

> **Note:**\
> [Zod](https://zod.dev/) is optional but highly recommended.\
> Writing validators manually is error-prone, and skipping validation means losing early guarantees about what your auth server provides. You can use another validator though, it doesn't have to be Zod.

## Required wiring

### Disable SSR

Set `ssr: false` in `nuxt.config.ts`.

{% code title="nuxt.config.ts" %}
```ts
export default defineNuxtConfig({
    ssr: false
});
```
{% endcode %}

### Enable the Nuxt module

Add `"oidc-spa/nuxt-spa"` to `modules`.

In the example, this happens in [`nuxt.config.ts`](https://github.com/keycloakify/oidc-spa/blob/main/examples/nuxt-spa/nuxt.config.ts).

### Create a client plugin

Provide `$oidc` from a client-only plugin.

In the example, this lives in [`app/plugins/01.oidc.client.ts`](https://github.com/keycloakify/oidc-spa/blob/main/examples/nuxt-spa/app/plugins/01.oidc.client.ts).

That plugin uses either `createOidc()` for a real provider or `createMockOidc()` for mock mode.

### Expose auth helpers with a composable

The example wraps `$oidc` in [`app/composables/useAuth.ts`](https://github.com/keycloakify/oidc-spa/blob/main/examples/nuxt-spa/app/composables/useAuth.ts).

It exposes helpers like `login`, `logout`, `register`, and `fetchWithAuth`.

It also handles the auto-logout countdown subscription.

### Protect pages with route middleware

Use route middleware for guarded pages.

In the example, [`app/middleware/auth.ts`](https://github.com/keycloakify/oidc-spa/blob/main/examples/nuxt-spa/app/middleware/auth.ts) redirects unauthenticated users to login and supports role checks through route meta.

### Runtime config

The example reads these public runtime config values:

* `oidcIssuerUri`
* `oidcClientId`
* `oidcUseMock`

You usually provide them through `.env.local`.

## Routes in the example

* `/` public page
* `/protected` guarded page
* `/admin-only` guarded page with a role check

## Reusing this pattern in your app

At a high level:

1. Set `ssr: false`.
2. Add `"oidc-spa/nuxt-spa"` to `modules`.
3. Create a client plugin that provides `$oidc`.
4. Add a `useAuth()` composable for app-level auth helpers.
5. Add route middleware for protected pages.

This example is intentionally opinionated and practical.

Use it as a starting point and adapt provider-specific options as needed.

{% include "../.gitbook/includes/creating-an-api-server.md" %}
