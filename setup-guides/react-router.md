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

* React Router "Framework" does not expose the abstractions nessesary to implement a full stack experience. If you want to use it with oidc-spa you need to enable SPA mode.
* Because React Router Framwork does not expose a true entrypoint you won't benefit from the same security guarenties you get with any other solution. oidc-spa is not any less secure than another client side OIDC client, but it's unique security feature do not apply here.

Use TanStack Start if you're picking it by default. You won't regret making the switch.
{% endhint %}

### Enabling SPA mode

As of today, to use oidc-spa you need to [enable SPA mode](https://reactrouter.com/how-to/spa).

<pre class="language-typescript" data-title="react-router.config.ts"><code class="lang-typescript">import type { Config } from "@react-router/dev/config";

export default {
<strong>    ssr: false
</strong>} satisfies Config;
</code></pre>

### Learning with the Example

{% embed url="https://example-react-router-framework.oidc-spa.dev/" %}

```bash
npx gitpick keycloakify/oidc-spa/tree/main/examples/react-router-framework rr-framework-oidc
cd rr-framework-oidc
cp .env.local.sample .env.local
yarn
yarn dev
```
{% endtab %}
{% endtabs %}
