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
This is for setting for integrating oidc-spa with react-router in [`Framework Mode`](https://reactrouter.com/start/modes). &#x20;

## Enabling SPA mode

As of today, to use oidc-spa you need to [enable SPA mode](https://reactrouter.com/how-to/spa).

<pre class="language-typescript" data-title="react-router.config.ts"><code class="lang-typescript">import type { Config } from "@react-router/dev/config";

export default {
    // Config options...
    // Server-side render by default, to enable SPA mode set this to `false`
<strong>    ssr: false
</strong>} satisfies Config;
</code></pre>

## oidc.client.ts&#x20;

Make sure you create a `app/oidc.client.ts` file, (instead of `app/oidc.ts`).

{% embed url="https://github.com/keycloakify/oidc-spa/blob/main/examples/react-router-framework/app/oidc.client.ts" %}
Example of oidc.client.ts file
{% endembed %}

## Setting up the entrypoint

Create thoses two files:

{% embed url="https://github.com/keycloakify/oidc-spa/blob/main/examples/react-router-framework/app/entry.client.tsx" %}
app/entry.client.tsx
{% endembed %}

{% embed url="https://github.com/keycloakify/oidc-spa/blob/main/examples/react-router-framework/app/entry.client.lazy.tsx" %}
app/entry.client.lazy.tsx
{% endembed %}

## Working with loaders

{% hint style="success" %}
If your whole app requires user to be authenticated ([autoLogin: true](../auto-login.md)) you can skip this section. &#x20;
{% endhint %}

The default approach when you want to enforce that the user be logged in when accesing a given route is to wrap the component into `withLoginEnforced()`, example: &#x20;

{% code title="pages/invoices.tsx" %}
```tsx
import { useState, useEffect } from "react";
import { withLoginEnforced, fetchWithAuth } from "../oidc.client";

const Invoices = withLoginEnforced(
    () => {
        const [invoices, setInvoices] = useState<Invoice[] | undefined>(undefined);

        useEffect(() => {
            fetchWithAuth("/api/invoices")
                .then(r => r.json())
                .then(setInvoices);
        }, []);

        if (invoices === undefined) {
            return <div>Loading invoices...</div>;
        }

        return (
            <div>
                {invoices.map(invoice => (
                    <div key={invoice.id}>{invoice.amount}</div>
                ))}
            </div>
        );
    },
    {
        onRedirecting: () => <div>Redirecting to login...</div>
    }
);

export default Invoices;
```
{% endcode %}

This approach is framework agnostic and always works however, you might want to use the loaders to doload the data, for that you would use `enforceLogin()` istead of `withLoginEnforced`:

<pre class="language-tsx" data-title="pages/invoices.tsx"><code class="lang-tsx"><strong>import { enforceLogin, fetchWithAuth } from "../oidc.client";
</strong>import type { Route } from "./+types/invoices";
import { useLoaderData } from "react-router";

export async function clientLoader(params: Route.ClientLoaderArgs) {
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

## Running the example

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
