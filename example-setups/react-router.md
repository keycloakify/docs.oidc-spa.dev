---
icon: route
---

# React Router

{% tabs %}
{% tab title="Declarative or Data Mode" %}
I will redact this later
{% endtab %}

{% tab title="Framwork Mode" %}
This is for setting for integrating oidc-spa with react-router in [`Framwork Mode`](https://reactrouter.com/start/modes). &#x20;

## Enabling SPA mode

oidc-spa, at least for now, is a library for Single Page Applications. Meaning that the distibution of the App must be generated statically and the components cannot be rendered server side. &#x20;

That being said you can use React Router in Framork mode by enabeling the SPA mode. &#x20;

<pre class="language-typescript" data-title="react-router.config.ts"><code class="lang-typescript">import type { Config } from "@react-router/dev/config";

export default {
    // Config options...
    // Server-side render by default, to enable SPA mode set this to `false`
<strong>    ssr: false
</strong>} satisfies Config;
</code></pre>

## oidc.client.ts&#x20;

Even in SPA mode, react-router in framwork mode will prerender the app at build time.  \
So you have to let it know that oidc-spa is meant for the browser by using the .client.ts file extention.\
So instead of creating an app/oidc.ts file, create an app/oidc.client.ts file.

## Setting up the entrypoint

Configure the entrypoint as instructed in the instalation guide (tab React-router Framwork mode):

{% content-ref url="../" %}
[..](../)
{% endcontent-ref %}

## Working with loaders

{% hint style="success" %}
If your whole app requires user to be authenticated ([autoLogin: true](../auto-login.md)) you can skip this section. &#x20;
{% endhint %}

The default approach when you want to enforce that the user be logged in when accesing a given route is to wrap the component into withLoginEnforced(), example: &#x20;

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

This approach is framwork agnostic and always work however, you might want to use the loaders to doload the data, for that you would use enforceLogin() istead of withLoginEnforced:

<pre class="language-tsx" data-title="pages/invoices.tsx"><code class="lang-tsx"><strong>import { enforceLogin, fetchWithAuth } from "../oidc.client";
</strong>import type { Route } from "./+types/invoices";
import { useLoaderData } from "react-router";

export async function clientLoader({ request }: Route.ClientLoaderArgs) {
<strong>    await enforceLogin(request.url);
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
