---
icon: route
---

# React Router

{% hint style="info" %}
NOTE: This is a react-router setup in [declarative mode](https://reactrouter.com/start/declarative/installation).\
For framwork mode, refer to [this issue](https://github.com/keycloakify/oidc-spa/issues/59#issuecomment-2735197918).\
We will come up with an example soon.
{% endhint %}

The example setup is live here: [https://example-react-router.oidc-spa.dev/](https://example-react-router.oidc-spa.dev/)

Run it locally with:

```bash
npx degit https://github.com/keycloakify/oidc-spa/examples/react-router oidc-spa-react-router
cd oidc-spa-react-router
cp .env.local.sample .env.local
yarn
yarn dev
```

{% embed url="https://github.com/keycloakify/oidc-spa/tree/main/examples/react-router" %}
Source code
{% endembed %}

## Working with loaders





\
If there are some pages of your app that can be browser anonymously and some other that requires authentication the standard way to enforce authentication is to use the withLoginEnforced() higher order component: &#x20;

```tsx
import { useLoaderData } from "react-router";
import { fetchWithAuth, enforceLogin } from "../oidc";

export async function clientLoader({ request }: Route.ClientLoaderArgs) {
    await enforceLogin(request.url);

    return fetchWithAuth("/api/invoices").then(r => r.json());
}


export default function Invoices() {
  let invoices = useLoaderData<typeof clientLoader>();
  // ...
}
```





{% tabs %}
{% tab title="Declarative or Data Mode" %}

{% endtab %}

{% tab title="Framwork Mode" %}
This is for setting for integrating oidc-spa with react-router in `Framwork Mode`. &#x20;

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


{% endtab %}
{% endtabs %}









