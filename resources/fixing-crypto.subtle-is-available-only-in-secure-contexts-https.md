---
icon: file-lock
---

# Fixing Crypto.subtle is available only in secure contexts (HTTPS)

oidc-spa internally makes use of the Ctypto.subtle browser API.  \
This builtin is only available when your app is served in HTTPS.  \
However for some intratnet usecase, with local DNS or static IPs setting up HTTPs might not be feasable.  \
\
You can work around this problem by using this polyfill:  \
\
webcrypto-liner: [https://www.npmjs.com/package/webcrypto-liner](https://www.npmjs.com/package/webcrypto-liner)  \
\
You should first install the lib in your porject:  \
\
npm install --save webcrypto-liner

Then edit your public.html (or the place where you can edit what's present in the head if you're using TanStack Start or ReactRouter in Framwork mode)\
\
And add thoses lines:  \


```html
<head>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/babel-polyfill/7.7.0/polyfill.min.js"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/asmCrypto/2.3.2/asmcrypto.all.es5.min.js"></script>
  <script src="https://cdn.rawgit.com/indutny/elliptic/master/dist/elliptic.min.js">
</head>
```

Then you must make sure to import webcrypto-liner just before oidc-spa/entrypoint

```typescript
import "webcrypto-liner/build/webcrypto-liner.shim";
import { oidcEarlyInit } from "oidc-spa/entrypoint";

const { shouldLoadApp } = oidcEarlyInit({
    freezeFetch: true,
    freezeXMLHttpRequest: true
});

if (shouldLoadApp) {
    import("./client.lazy");
}
```
