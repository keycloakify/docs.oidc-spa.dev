---
icon: route
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/UhNOMoIddws1XoAnT5Nn/integration-guides/react-router
---

# React Router

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

{% tabs %}
{% tab title="Vite Plugin" %}
If you’re not using Vite and you can’t edit your app’s entry file, run `oidcEarlyInit()` in the `src/oidc.ts`.

<pre class="language-typescript" data-title="vite.config.ts"><code class="lang-typescript">import { defineConfig } from "vite";
<strong>import { oidcSpa } from "oidc-spa/vite-plugin";
</strong>
export default defineConfig({
    plugins: [
        // ...
<strong>        oidcSpa()
</strong>    ]
});
</code></pre>
{% endtab %}

{% tab title="Manual - Recommended" %}
Pick this approach if:

* You're not in a Vite project and
* Your app has a single client entrypoint.

***

Let's assume your app entrypoint is `src/main.ts`.

First, rename it to `src/main.lazy.ts`.

```bash
mv src/main.ts src/main.lazy.ts
```

Then create a new `src/main.ts` file:

{% code title="src/main.ts" %}
```typescript
import { oidcEarlyInit } from "oidc-spa/entrypoint";

const { shouldLoadApp } = oidcEarlyInit({
    BASE_URL: "/" // The path where your app is hosted. You can also pass it later to createOidc().
});

if (shouldLoadApp) {
    // Note: Deferring the main app import adds a few milliseconds to cold start,
    // but dramatically speeds up auth. Overall, it's a net win.
    import("./main.lazy");
}
```
{% endcode %}
{% endtab %}

{% tab title="Manual - Easy" %}
If you’re not using Vite and you can’t edit your app’s entry file, run `oidcEarlyInit()` in the same module where you call `createOidc()`.

Note however that implementing this option [dowgrades the security posture of your app](../security-features/overview.md#how-oidc-spa-achieves-this-in-a-nutshell) compared to the two other approaches, and, in some instances, might conflict with your client side routing library.

<pre class="language-typescript" data-title="src/oidc.ts"><code class="lang-typescript"><strong>import { oidcEarlyInit, } from "oidc-spa/entrypoint";
</strong>import { oidcSpa } from "oidc-spa/react-spa";

<strong>// Should run as early as possible.  
</strong><strong>oidcEarlyInit({ 
</strong><strong>   BASE_URL: "/" // The path where your app is hosted
</strong><strong>});
</strong>
export const { /* ... */ } = oidcSpa./*...*/
</code></pre>
{% endtab %}
{% endtabs %}

## Learning from the example

You're going to be cloning this example:

{% embed url="https://example-react-router-framework.oidc-spa.dev/" %}

React Router v7 has [three modes](https://reactrouter.com/start/modes) pick the one for you:&#x20;

{% tabs %}
{% tab title="Declarative Mode" %}
```bash
npx gitpick keycloakify/oidc-spa/tree/main/examples/react-router-declarative rr-declarative-oidc
cd rr-declarative-oidc
# You can use our preconfigured Keycloak, Auth0, or Google OAuth test accounts
cp .env.local.sample .env.local
npm install
npm run dev

# Start exploring with: src/oidc.ts
```

{% embed url="https://github.com/keycloakify/oidc-spa/tree/main/examples/react-router-declarative" %}
{% endtab %}

{% tab title="Data Mode" %}
```bash
npx gitpick keycloakify/oidc-spa/tree/main/examples/react-router-data rr-data-oidc
cd rr-data-oidc
# You can use our preconfigured Keycloak, Auth0, or Google OAuth test accounts
cp .env.local.sample .env.local
npm install
npm run dev

# Start exploring with: src/oidc.ts
```

{% embed url="https://github.com/keycloakify/oidc-spa/tree/main/examples/react-router-data" %}
{% endtab %}

{% tab title="Framework Mode" %}
### Enabling SPA mode

This is non optional. React Router Framework does not expose the primitives to enable solution like oidc-spa to provide a full stack story. (You may want to give [TanStack Start](https://tanstack.com/start/latest) a try)

<pre class="language-typescript" data-title="react-router.config.ts"><code class="lang-typescript">import type { Config } from "@react-router/dev/config";

export default {
<strong>    ssr: false
</strong>} satisfies Config;
</code></pre>

### The example

```bash
npx gitpick keycloakify/oidc-spa/tree/main/examples/react-router-framework rr-framework-oidc
cd rr-framework-oidc
# You can use our preconfigured Keycloak, Auth0, or Google OAuth test accounts
cp .env.local.sample .env.local
npm install
npm run dev

# Start exploring with: src/oidc.ts
```

{% embed url="https://github.com/keycloakify/oidc-spa/tree/main/examples/react-router-framework" %}
{% endtab %}
{% endtabs %}

{% include "../.gitbook/includes/creating-an-api-server.md" %}
