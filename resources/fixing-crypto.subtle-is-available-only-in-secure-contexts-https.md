---
hidden: true
icon: file-lock
---

# Fixing Crypto.subtle is available only in secure contexts (HTTPS)

{% hint style="success" %}
Starting with oidc-spa 8.6.18, polyfills for Crypto.subtle are automatically loaded when needed.&#x20;

Nothing to do, evrything will work even if your app is deployed without SSL.
{% endhint %}

`oidc-spa` internally relies on the [`Crypto.subtle`](https://developer.mozilla.org/en-US/docs/Web/API/SubtleCrypto) browser API for cryptographic operations.\
This API is only available when your app is served over **HTTPS** or from **localhost**.

However, in certain **intranet** environments, for example, when using a local DNS entry or static IP, setting up HTTPS might not be feasible.

In those cases, you can work around the issue by installing a polyfill such as [**webcrypto-liner**](https://www.npmjs.com/package/webcrypto-liner).

***

## 1. Install the polyfill

```bash
npm install --save webcrypto-liner-shim
```

***

## 3. Import the shim

{% tabs %}
{% tab title="Framework Agnostic" %}
<pre class="language-typescript" data-title="src/oidc.ts"><code class="lang-typescript">import { createOidc } from "oidc-spa/core";

<strong>if( crypto.subtle === undefined ){
</strong><strong>    await import("webcrypto-liner-shim");
</strong><strong>}
</strong>
const oidc = await createOidc({
    issuerUri: "...",
    clientId: "..."
});
</code></pre>
{% endtab %}

{% tab title="React" %}
{% code title="src/oidc.ts" %}
```typescript
import { oidcSpa } from "oidc-spa/react-spa";

export const {
    bootstrapOidc,
    //...
} = oidcSpa
    .withExpectedDecodedIdTokenShape({ /*...*/ })
    .createUtils();

(async ()=> {

    if( crypto.subtle === undefined ){
        await import("webcrypto-liner-shim");
    }

    bootstrapOidc({
          implementation: "real",
          issuerUri: import.meta.env.VITE_OIDC_ISSUER_URI,
          clientId: import.meta.env.VITE_OIDC_CLIENT_ID
    });

})();

// ...

```
{% endcode %}
{% endtab %}
{% endtabs %}
