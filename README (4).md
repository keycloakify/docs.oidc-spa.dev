---
icon: up
---

# v9 -> v10

{% tabs %}
{% tab title="Vite Plugin" %}
{% code title="vite.config.ts" %}
```diff
  oidcSpa({
    browserRuntimeFreeze: { enabled: true },
    tokenSubstitution: {
        enabled: true,
-       trustedThirdPartyResourceServers: ["s3.amazonaws.com"],
+       trustedExternalResourceServers: [
+           "*.{{location.hostname.split('.').slice(-2).join('.')}}",
+           "s3.amazonaws.com"
+       ]
    },
+   DPoP: { mode: "auto" /* or "enforced" */}
});
```
{% endcode %}

```diff
 createOidc({ // or bootstrapOidc({
     // ...
-    dpop: "auto"
 })     
```
{% endtab %}

{% tab title="Manual" %}
{% code title="src/main.ts" %}
```diff
 import { oidcEarlyInit } from "oidc-spa/entrypoint";
-import { enableTokenSubstitution } from "oidc-spa/token-substitution";
+import { browserRuntimeFreeze } from 'oidc-spa/browser-runtime-freeze';
+import { DPoP } from 'oidc-spa/DPoP';
+import { tokenSubstitution } from 'oidc-spa/token-substitution';
 
 const { shouldLoadApp } = oidcEarlyInit({
-   browserRuntimeFreeze: { enabled: true },
-   extraDefenseHook: () => {
-       enableTokenSubstitution({
-           trustedThirdPartyResourceServers: ["s3.amazonaws.com"]
-       });
-   }
+  securityDefenses: {
+    ...browserRuntimeFreeze({
+      //exclude: [ "fetch", "XMLHttpRequest", "Promise"]
+    }),
+    ...DPoP({ mode: 'auto' }),
+    ...tokenSubstitution({
+      trustedExternalResourceServers: [
+        "s3.amazonaws.com",
+        `*.${location.hostname.split('.').slice(-2).join('.')}`,
+      ],
+    }),
+  }
 });
```
{% endcode %}

```diff
 createOidc({ // or bootstrapOidc({
     // ...
-    dpop: "auto"
 })     
```
{% endtab %}
{% endtabs %}

Takeways:&#x20;

* trustedThirdPartyResourceServers renamed to trustedExternalResourceServers
* If you want trust same site origins (\*.my-domain.com) you should state it explicitely with  `"*.{{location.hostname.split('.').slice(-2).join('.')}}"` (Previously it was enabled by default).
* DPoP is now globally enabled, not on a per OIDC client instance basis.
