---
icon: route
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
{% tab title="Vite" %}
In Vite apps, this is done through a Vite Plugin (If you'd rather avoid using the Vite plugin checkout the Other SPAs tab).

<pre class="language-typescript" data-title="vite.config.ts"><code class="lang-typescript">import { defineConfig } from "vite";
<strong>import { oidcSpa } from "oidc-spa/vite-plugin";
</strong>
export default defineConfig({
    plugins: [
        // ...
<strong>        oidcSpa({
</strong><strong>            freezeFetch: true,
</strong><strong>            freezeXMLHttpRequest: true,
</strong><strong>            freezeWebSocket: true
</strong><strong>        })
</strong>    ]
});
</code></pre>
{% endtab %}

{% tab title="Other" %}
First rename your entry point file from `main.tsx` (or `main.ts` or whatever it is) to `main.lazy.tsx`

```bash
mv src/main.ts src/main.lazy.ts
```

Then create a new `index.tsx` file:

{% code title="src/index.tsx" %}
```typescript
import { oidcEarlyInit } from "oidc-spa/entrypoint";

const { shouldLoadApp } = oidcEarlyInit({
    freezeFetch: true,
    freezeXMLHttpRequest: true,
    freezeWebSocket: true,
    BASE_URL: "/" // The path where your app is hosted, can also be provided later to createOidc()
});

if (shouldLoadApp) {
    // Note: Deferring the main app import adds a few milliseconds to cold start,
    // but dramatically speeds up auth. Overall, it's a net win.
    import("./index.lazy");
}
```
{% endcode %}

If you don't have a precise entrypoint that you can simply override, just call oidcEarlyInit as soon as possible and try canceling as much work as possible when `shouldLoadApp` is false.
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

{% endtab %}

{% tab title="Framework Mode" %}
{% hint style="warning" %}
IMPORTANT NOTICE:

Because React Router Framwork does not expose a true entrypoint you won't benefit from the same security guarenties you get with any other solution. oidc-spa is not any less secure than another client side OIDC client, but it's unique security caims do not apply here.
{% endhint %}

### Enabling SPA mode

This is non optional. React Router Framework does not expose the primitives to enable solution like oidc-spa to provide a full stack story. (You may want to give [TanStack Start](https://tanstack.com/start/latest) a try, it's the same thing than React Router Framwork but better.w)

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

## Creating an API server

Now that authentication is handled, there’s one last piece of the puzzle: your resource server, the backend your app will communicate with.

This can be any type of service: a REST API, tRPC server, or WebSocket endpoint, as long as it can validate access tokens issued by your IdP.

If you’re building it in JavaScript or TypeScript (for example, using Express), oidc-spa provides ready-to-use utilities to decode and validate access tokens on the server side.

You’ll find the full documentation here:

{% content-ref url="../integration-guides/tanstack-router-+-node-rest-api.md" %}
[tanstack-router-+-node-rest-api.md](../integration-guides/tanstack-router-+-node-rest-api.md)
{% endcontent-ref %}
