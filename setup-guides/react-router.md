---
icon: route
---

# React Router

{% tabs %}
{% tab title="Declarative or Data Mode" %}
The example setup is live here: [https://example-react-router.oidc-spa.dev/](https://example-react-router-framework.oidc-spa.dev/)

Run it locally with:

```bash
npx degit https://github.com/keycloakify/oidc-spa/examples/react-router oidc-spa-react-router
cd oidc-spa-react-router
cp .env.local.sample .env.local
yarn
yarn dev
```

{% embed url="https://github.com/keycloakify/oidc-spa/tree/main/examples/react-router" %}
{% endtab %}

{% tab title="Framework Mode" %}
This is for setting for integrating oidc-spa with react-router in [`Framework Mode`](https://reactrouter.com/start/modes).

{% hint style="warning" %}
IMPORTANT NOTICE:

* React Router "Framework" does not expose the abstractions nessesary to implement a full stack auth experience. If you want to use it with oidc-spa you need to enable SPA mode.
* Because React Router Framwork does not expose a true entrypoint you won't benefit from the same security guarenties you get with any other solution. oidc-spa is not any less secure than another client side OIDC client, but it's unique security feature do not apply here.

TL,DR: React Router is a hard sell compared to SanStack Router/Start.
{% endhint %}

### Instaling

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
> Writing validators manually is error-prone, and skipping validation means losing early guarantees about what your auth server provides.

### Enabling SPA mode

This is non optional.

<pre class="language-typescript" data-title="react-router.config.ts"><code class="lang-typescript">import type { Config } from "@react-router/dev/config";

export default {
<strong>    ssr: false
</strong>} satisfies Config;
</code></pre>

### Enabling the Vite Plugin

<pre class="language-typescript"><code class="lang-typescript">import { reactRouter } from "@react-router/dev/vite";
import { defineConfig } from "vite";
import tsconfigPaths from "vite-tsconfig-paths";
<strong>import { oidcSpa } from "oidc-spa/vite-plugin";
</strong>
export default defineConfig({
    plugins: [
        reactRouter(),
<strong>        oidcSpa(),
</strong>        tsconfigPaths()
    ]
});
</code></pre>

### Learning with the Example

{% embed url="https://example-react-router-framework.oidc-spa.dev/" %}

```bash
npx gitpick keycloakify/oidc-spa/tree/main/examples/react-router-framework rr-framework-oidc
cd rr-framework-oidc
# You can use our preconfigured Keycloak, Auth0, or Google OAuth test accounts
cp .env.local.sample .env.local
yarn
yarn dev
```
{% endtab %}
{% endtabs %}
