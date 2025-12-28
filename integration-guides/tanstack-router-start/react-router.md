---
icon: umbrella-beach
---

# TanStack Router

## Instaling

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

{% include "../../.gitbook/includes/setup-option.md" %}

## Learning from the example

You're going to be cloning this example:

{% embed url="https://example-tanstack-router.oidc-spa.dev/" %}

TanStack Router has [two modes](https://tanstack.com/router/latest/docs/framework/react/quick-start#new-project-setup) pick the one for you:&#x20;

{% tabs %}
{% tab title="File-Based Route Generation" %}
```bash
npx gitpick keycloakify/oidc-spa/tree/main/examples/tanstack-router-file-router tr-oidc
cd tr-oidc
# You can use our preconfigured Keycloak, Auth0, or Google OAuth test accounts
cp .env.local.sample .env.local
npm install
npm run dev

# Start exploring with: src/oidc.ts
```

{% embed url="https://github.com/keycloakify/oidc-spa/tree/main/examples/tanstack-router-file-router" %}
{% endtab %}

{% tab title="Code-Based Route Configuration" %}
> Comming Soon
{% endtab %}
{% endtabs %}

## Creating an API server

Now that authentication is handled, there’s one last piece of the puzzle: your resource server, the backend your app will communicate with.

This can be any type of service: a REST API, tRPC server, or WebSocket endpoint, as long as it can validate access tokens issued by your IdP.

If you’re building it in JavaScript or TypeScript (for example, using Express), oidc-spa provides ready-to-use utilities to decode and validate access tokens on the server side.

You’ll find the full documentation here:

{% content-ref url="../tanstack-router-+-node-rest-api/" %}
[tanstack-router-+-node-rest-api](../tanstack-router-+-node-rest-api/)
{% endcontent-ref %}
