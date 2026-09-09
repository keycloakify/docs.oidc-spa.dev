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

{% tabs %}
{% tab title="Vite Plugin" %}
If you're in a Vite project, the recomended approach is to use oidc-spa's Vite plugin.

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
If you’re not using Vite and you can’t edit your app’s entry file, run `oidcEarlyInit()` in the `src/oidc.ts`.

Note however that implementing this option [dowgrades the security posture of your app](../../security-features/overview.md#how-oidc-spa-achieves-this-in-a-nutshell) compared to the two other approaches, and, in some instances, might conflict with your client side routing library.

<pre class="language-typescript" data-title="src/oidc.ts"><code class="lang-typescript"><strong>import { oidcEarlyInit, } from "oidc-spa/entrypoint";
</strong>import { oidcSpa } from "oidc-spa/react-spa";

// Should run as early as possible.  
<strong>oidcEarlyInit({ 
</strong><strong>   BASE_URL: "/" // The path where your app is hosted
</strong><strong>});
</strong>
export const { /* ... */ } = oidcSpa./*...*/
</code></pre>
{% endtab %}
{% endtabs %}

## Learning from the example

You're going to be cloning this example:

{% embed url="https://example-tanstack-router.oidc-spa.dev/" %}

TanStack Router has [two modes](https://tanstack.com/router/latest/docs/framework/react/quick-start#new-project-setup) pick the one for you:

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

{% include "../../.gitbook/includes/creating-an-api-server.md" %}
