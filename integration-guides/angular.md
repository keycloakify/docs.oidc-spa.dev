---
icon: angular
---

# Angular

## Installation and Setup

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

To protect tokens against supply-chain attacks and XSS, oidc-spa must run some initialization code _before any other JavaScript in your app_.

This design provides much stronger security guarantees than any other adapter, and it also delivers unmatched login performance. More details [here](../why-oidcearlyinit.md).

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

## Basic Example

```bash
npx degit https://github.com/keycloakify/oidc-spa/examples/angular oidc-spa-angular
cd oidc-spa-angular
npm install
npm run start
```

{% embed url="https://github.com/keycloakify/oidc-spa/tree/main/examples/angular" %}

## Advanced example

Live here: [https://example-angular.oidc-spa.dev](https://example-angular.oidc-spa.dev/)

This setup show you how you can:&#x20;

* Early rendering of public pages before oidc has finished initializing.
* Mock implementation of the adapter.
* Fetching the initialization parameter remotly.
* Protecting groupes based on roles.
* Validating the shape of the access token.

```bash
npx degit https://github.com/keycloakify/oidc-spa/examples/angular-kitchensink oidc-spa-angular-kitchensink
cd oidc-spa-angular-kitchensink
npm install
npm run start
```

{% embed url="https://github.com/keycloakify/oidc-spa/tree/main/examples/angular-kitchensink" %}

## Creating an API server

Now that authentication is handled, there’s one last piece of the puzzle: your resource server, the backend your app will communicate with.

This can be any type of service: a REST API, tRPC server, or WebSocket endpoint, as long as it can validate access tokens issued by your IdP.

If you’re building it in JavaScript or TypeScript (for example, using Express), oidc-spa provides ready-to-use utilities to decode and validate access tokens on the server side.

You’ll find the full documentation here:

{% content-ref url="tanstack-router-+-node-rest-api.md" %}
[tanstack-router-+-node-rest-api.md](tanstack-router-+-node-rest-api.md)
{% endcontent-ref %}
