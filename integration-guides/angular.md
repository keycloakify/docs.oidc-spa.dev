---
icon: angular
---

# Angular

## Installation and Setup

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

> **Note:** [Zod](https://zod.dev/) is optional but highly recommended (it's not used in the simple example, only in the advanced one) .\
> Writing validators manually is error-prone, and skipping validation means losing early guarantees about what your auth server provides. You can use another validator though, it doesn't have to be Zod.

## Editing your entrypoint

To protect tokens against supply-chain attacks and XSS, oidc-spa must run some initialization code _before any other JavaScript in your app_.

This design provides much stronger security guarantees than any other adapter, and it also delivers unmatched login performance. More details [here](/broken/pages/o7mf9sx2j3zFyddFEHST).

First rename your entry point file from `main.ts` to `main.lazy.ts`

```bash
mv src/main.tsx src/main.lazy.tsx
```

Then create a new `main.ts` file:

{% code title="main.ts" %}
```typescript
import { oidcEarlyInit } from "oidc-spa/entrypoint";

const { shouldLoadApp } = oidcEarlyInit();

if (shouldLoadApp) {
    // Note: Deferring the main app import adds a few milliseconds to cold start,
    // but dramatically speeds up auth. Overall, it's a net win.
    import("./main.lazy");
}
```
{% endcode %}

***

## Basic Example

```bash
npx gitpick keycloakify/oidc-spa/tree/main/examples/angular oidc-spa-angular
cd oidc-spa-angular
npm install
npm run start
```

{% embed url="https://github.com/keycloakify/oidc-spa/tree/main/examples/angular" %}

## Advanced example

Live here: [https://example-angular.oidc-spa.dev](https://example-angular.oidc-spa.dev/)

This setup show you how you can:&#x20;

* Mock implementation of the adapter.
* Fetching the initialization parameter remotly.
* Protecting groupes based on roles.
* Validating the shape of the access token.
* Early rendering of public pages before oidc has finished initializing.

```bash
npx gitpick keycloakify/oidc-spa/tree/main/examples/angular-kitchensink oidc-spa-angular-kitchensink
cd oidc-spa-angular-kitchensink
npm install
npm run start
```

{% embed url="https://github.com/keycloakify/oidc-spa/tree/main/examples/angular-kitchensink" %}

{% include "../.gitbook/includes/creating-an-api-server.md" %}
