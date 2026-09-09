---
description: Use oidc-spa in a Next.js App Router app.
icon: triangle
---

# Next.js

## Before you start

{% hint style="warning" %}
Using `oidc-spa` in Next.js is easy and infra-light.

But it also changes the architecture.

Authentication happens in the browser. The server rendering your pages cannot know who the user is. That removes most SSR auth-at-render-time benefits and effectively pushes your app toward SPA behavior.

Use this setup only if you accept that trade-off.
{% endhint %}

## The example

{% embed url="https://youtu.be/zkOWKeTZcYk" %}

This example shows the minimum wiring needed for a Next.js App Router app.

```bash
npx gitpick keycloakify/oidc-spa/tree/main/examples/next oidc-spa-next
cd oidc-spa-next
cp .env.local.sample .env.local
npm install
npm run dev

# Start exploring with: lib/oidc.tsx
```

{% embed url="https://github.com/keycloakify/oidc-spa/tree/main/examples/next" %}

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

### Initialize early on the client

Use `instrumentation-client.ts` to run `oidcEarlyInit()` as early as possible.

{% code title="instrumentation-client.ts" %}
```ts
import { oidcEarlyInit } from "oidc-spa/entrypoint";

oidcEarlyInit({
    BASE_URL: process.env.__NEXT_ROUTER_BASEPATH || "/"
});
```
{% endcode %}

### Wrap the app

Wrap the whole app in `OidcInitializationGate`.

In the example, this happens in [`app/layout.tsx`](https://github.com/keycloakify/oidc-spa/blob/main/examples/next/app/layout.tsx).

### Add Next-specific adapters

The `OidcInitializationGate` and `withLoginEnforced` utils exported by oidc-spa cannot be used directly in Next.js as-is.

They need Next-specific adapters.

In the example, that adaptation lives in [`lib/oidc.tsx`](https://github.com/keycloakify/oidc-spa/blob/main/examples/next/lib/oidc.tsx).

### Keep oidc-spa on the client

Anything that touches `oidc-spa` must run on the client.

Add `"use client";` to those modules.

## Important limitation

Next.js cannot know who the user is at render time when auth is handled by `oidc-spa` in the browser.

In practice, that means you give up most SSR auth-at-render-time benefits.

You should treat this setup as a SPA architecture running inside Next.js.

## Routes in the example

* `/` public landing page with login and logout controls
* `/protected` guarded page using the custom Next-compatible `withLoginEnforced`
* `/admin-only` guarded page with a simple role check

{% include "../.gitbook/includes/creating-an-api-server.md" %}
