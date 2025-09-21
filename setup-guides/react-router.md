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

### Enabling SPA mode

As of today, to use oidc-spa you need to [enable SPA mode](https://reactrouter.com/how-to/spa).

<pre class="language-typescript" data-title="react-router.config.ts"><code class="lang-typescript">import type { Config } from "@react-router/dev/config";

export default {
<strong>    ssr: false
</strong>} satisfies Config;
</code></pre>

### (Optional) Improve dev mode by pre-optimizing dependencies

In dev mode, listing your dependencies in `optimizeDeps.include` can prevent annoying glitches that forces you to reload your page manually.

<pre class="language-typescript" data-title="vite.config.ts"><code class="lang-typescript">import { reactRouter } from "@react-router/dev/vite";
import { defineConfig } from "vite";

export default defineConfig({
    plugins: [reactRouter()],
<strong>    optimizeDeps: {
</strong><strong>        include: [
</strong><strong>            "oidc-spa/react",
</strong><strong>            "oidc-spa/entrypoint",
</strong><strong>            "oidc-spa/tools/parseKeycloakIssuerUri",
</strong><strong>            "oidc-spa/tools/decodeJwt",
</strong><strong>            "zod"
</strong><strong>        ]
</strong><strong>    }
</strong>});
</code></pre>

### oidc.client.ts

Make sure you create a `app/oidc.client.ts` file, (instead of `app/oidc.ts`).

{% embed url="https://github.com/keycloakify/oidc-spa/blob/main/examples/react-router-framework/app/oidc.client.ts" %}
Example of oidc.client.ts file
{% endembed %}

### Setting up the entrypoint

Create thoses two files:

{% embed url="https://github.com/keycloakify/oidc-spa/blob/main/examples/react-router-framework/app/entry.client.tsx" %}
app/entry.client.tsx
{% endembed %}

{% embed url="https://github.com/keycloakify/oidc-spa/blob/main/examples/react-router-framework/app/entry.client.lazy.tsx" %}
app/entry.client.lazy.tsx
{% endembed %}

### Protecting pages and using the Page Loaders

Let's say we have an invoice page and you want this page to be accessible only when the user is logged in.  \
You also want to fetch the API to get the invoces of the user.  \
This is how you would implement it: &#x20;

<pre class="language-tsx" data-title="pages/invoices.tsx"><code class="lang-tsx"><strong>import { enforceLogin, fetchWithAuth } from "../oidc.client";
</strong>import type { Route } from "./+types/invoices";
import { useLoaderData } from "react-router";

export async function clientLoader(params: Route.ClientLoaderArgs) {
    // If you have `autoLogin: true` this isn't nessesary.  
<strong>    await enforceLogin(params);
</strong>    // If we are here, the user is logged in.
    const invoices = await fetchWithAuth("/api/invoices").then(r => r.json());
    return invoices;
}

export function HydrateFallback() {
    return &#x3C;div>Loading invoices...&#x3C;/div>;
}

export default function Invoices() {
    const invoices = useLoaderData&#x3C;typeof clientLoader>();

    return (
        &#x3C;div>
            {invoices.map(invoice => (
                &#x3C;div key={invoice.id}>{invoice.amount}&#x3C;/div>
            ))}
        &#x3C;/div>
    );
}
</code></pre>

### Running the example

The example setup is live here: [https://example-react-router-framework.oidc-spa.dev/](https://example-react-router-framework.oidc-spa.dev/)

Run it locally with:

```bash
npx degit https://github.com/keycloakify/oidc-spa/examples/react-router-framework oidc-spa-react-router
cd oidc-spa-react-router
cp .env.local.sample .env.local
yarn
yarn dev
```

{% embed url="https://github.com/keycloakify/oidc-spa/tree/main/examples/react-router-framework" %}
{% endtab %}
{% endtabs %}
