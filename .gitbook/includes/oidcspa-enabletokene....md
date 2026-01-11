---
title: oidcSpa({-   enableTokenE...
---

{% tabs %}
{% tab title="Vite Plugin" %}
{% code title="vite.config.ts" %}
```diff

  oidcSpa({
-   enableTokenExfiltrationDefense: true,
-   resourceServersAllowedHostnames: ["s3.amazonaws.com"],
+   browserRuntimeFreeze: { enabled: true },
+   tokenSubstitution: {
+       enabled: true,
+       trustedThirdPartyResourceServers: ["s3.amazonaws.com"]
+   }
});
```
{% endcode %}

If you’re migrating from the older `freeze*` flags:

{% code title="vite.config.ts" %}
```diff
 oidcSpa({
-    freezeFetch: false,
-    freezeXMLHttpRequest: false,
-    freezeWebSocket: true,
-    freezePromise: true
     // Add to `exclude` the APIs for which you had `freezeXxx: false`.
+    browserRuntimeFreeze: {
+        enabled: true,
+        exclude: ["fetch", "XMLHttpRequest"]
+    }
});
```
{% endcode %}
{% endtab %}

{% tab title="Manual" %}
{% code title="src/main.ts" %}
```diff
 import { oidcEarlyInit } from "oidc-spa/entrypoint";
+import { enableTokenSubstitution } from "oidc-spa/token-substitution";
 
 const { shouldLoadApp } = oidcEarlyInit({
-   enableTokenExfiltrationDefense: true,
-   resourceServersAllowedHostnames: ["s3.amazonaws.com"],
+   browserRuntimeFreeze: { enabled: true },
+   extraDefenseHook: () => {
+       enableTokenSubstitution({
+           trustedThirdPartyResourceServers: ["s3.amazonaws.com"]
+       });
+   }
 });
```
{% endcode %}

If you’re migrating from the older `freeze*` flags:

{% code title="src/main.ts" %}
```diff
 oidcEarlyInit({
-    freezeFetch: false,
-    freezeXMLHttpRequest: false,
-    freezeWebSocket: true,
-    freezePromise: true
     // Add to `exclude` the APIs for which you had `freezeXxx: false`.
+    browserRuntimeFreeze: {
+        enabled: true,
+        exclude: ["fetch", "XMLHttpRequest"]
+    }
});
```
{% endcode %}
{% endtab %}
{% endtabs %}
