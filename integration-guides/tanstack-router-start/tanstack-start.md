---
icon: umbrella-beach
---

# TanStack Start

## The Example/Tutorial

You'll be cloning this example, it's a very lightly modified version of the Starter that you get with `pnpm create @tanstack/start` where we've enabled autnetication for the todo list.

{% embed url="https://example-tanstack-start.oidc-spa.dev/" %}

```bash
npx gitpick keycloakify/oidc-spa/tree/main/examples/tanstack-start start-oidc
cd start-oidc
npm install
npm run dev

# By default, the example runs against Keycloak.
# You can edit the .env file to test other providers.
```

{% embed url="https://github.com/keycloakify/oidc-spa/tree/main/examples/tanstack-start" %}

***

## Instalations

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

***

Add the plugin to your `vite.config.ts`:

<pre class="language-typescript"><code class="lang-typescript">import { defineConfig } from "vite";
import { tanstackStart } from "@tanstack/react-start/plugin/vite";
import viteReact from "@vitejs/plugin-react";
import viteTsConfigPaths from "vite-tsconfig-paths";
import tailwindcss from "@tailwindcss/vite";
<strong>import { oidcSpa } from "oidc-spa/vite-plugin";
</strong>
export default defineConfig({
  plugins: [
    viteTsConfigPaths({ projects: ["./tsconfig.json"] }),
    tailwindcss(),
    tanstackStart(),
<strong>    oidcSpa({
</strong><strong>        freezeFetch: true,
</strong><strong>        freezeXMLHttpRequest: true,
</strong><strong>        freezeWebSocket: true
</strong><strong>    }),
</strong>    viteReact(),
  ],
});
</code></pre>

***

## Usage

You should be able to learn everything there is to know by checkout out [the example](tanstack-start.md)!  \
You're journey will start by looking at `src/oidc.ts`.
