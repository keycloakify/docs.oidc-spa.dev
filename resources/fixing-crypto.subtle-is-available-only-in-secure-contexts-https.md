---
hidden: true
icon: file-lock
---

# Fixing Crypto.subtle is available only in secure contexts (HTTPS)

`oidc-spa` internally relies on the [`Crypto.subtle`](https://developer.mozilla.org/en-US/docs/Web/API/SubtleCrypto) browser API for cryptographic operations.\
This API is only available when your app is served over **HTTPS** or from **localhost**.

However, in certain **intranet** environments, for example, when using a local DNS entry or static IP, setting up HTTPS might not be feasible.

In those cases, you can work around the issue by installing a polyfill such as [**webcrypto-liner**](https://www.npmjs.com/package/webcrypto-liner).

***

## 1. Install the polyfill

```bash
npm install --save webcrypto-liner
```

***

## 2. Add required scripts to your HTML head

Edit your `public.html` (or the file that defines your HTML head, e.g. in TanStack Start or React Router framework mode) and add the following scripts:

```html
<head>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/babel-polyfill/7.7.0/polyfill.min.js"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/asmCrypto/2.3.2/asmcrypto.all.es5.min.js"></script>
  <script src="https://cdn.rawgit.com/indutny/elliptic/master/dist/elliptic.min.js"></script>
</head>
```

***

## 3. Import the shim

{% code title="src/oidc.ts" %}
```typescript
import "webcrypto-liner/build/webcrypto-liner.shim";
// ...
```
{% endcode %}

***

✅ **Summary:**\
If you see the error `Crypto.subtle is available only in secure contexts (HTTPS)` in a non-HTTPS environment, install `webcrypto-liner`. This allows `oidc-spa` to work even on local or intranet setups without HTTPS.
