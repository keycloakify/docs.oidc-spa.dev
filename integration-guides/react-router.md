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

Next you need to enable the oidc-spa Vite Plugin: &#x20;

<pre class="language-typescript" data-title="vite.config.ts"><code class="lang-typescript">import { reactRouter } from "@react-router/dev/vite";
import { defineConfig } from "vite";
import tsconfigPaths from "vite-tsconfig-paths";
<strong>import { oidcSpa } from "oidc-spa/vite-plugin";
</strong>
export default defineConfig({
    plugins: [
        reactRouter(),
<strong>        oidcSpa({
</strong><strong>            freezeFetch: true,
</strong><strong>            freezeXMLHttpRequest: true,
</strong><strong>            freezeWebSocket: true
</strong><strong>        }),
</strong>        tsconfigPaths()
    ]
});
</code></pre>

## Suspense is optional

When running the example, you may notice that the page renders immediately, and then the components that depend on authentication (via `useOidc()`) appear a few milliseconds later.\
This is achieved by placing [**Suspense boundaries**](https://github.com/keycloakify/oidc-spa/blob/ce23b6b164f913de244e952a144e6ddae20d5e8a/examples/react-router-framework/app/components/Header.tsx#L42-L44) around components that call `useOidc()`.

{% embed url="https://youtu.be/tLk81s5JpbQ" %}

It’s important to understand that **this behavior is completely optional**.

If you prefer to avoid layout shifts and would rather render the page only once everything is ready, simply remove the Suspense boundaries.\
React will then fall back to the **nearest Suspense boundary** (or [the `HydrateFallback` component](https://github.com/keycloakify/oidc-spa/blob/ce23b6b164f913de244e952a144e6ddae20d5e8a/examples/react-router-framework/app/root.tsx#L40-L42) in framework mode).

Even without any Suspense boundaries at all, everything will still work correctly, React will just wait for the OIDC initialization process to complete before rendering your app.

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
yarn
yarn dev
```

{% embed url="https://github.com/keycloakify/oidc-spa/tree/main/examples/react-router-framework" %}
{% endtab %}
{% endtabs %}

## Creating an API server

Now that authentication is handled, there’s one last piece of the puzzle: your resource server, the backend your app will communicate with.

This can be any type of service: a REST API, tRPC server, or WebSocket endpoint, as long as it can validate access tokens issued by your IdP.

If you’re building it in JavaScript or TypeScript (for example, using Express), oidc-spa provides ready-to-use utilities to decode and validate access tokens on the server side.

You’ll find the full documentation here:

{% content-ref url="tanstack-router-+-node-rest-api.md" %}
[tanstack-router-+-node-rest-api.md](tanstack-router-+-node-rest-api.md)
{% endcontent-ref %}
