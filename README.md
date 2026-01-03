---
icon: up
---

# v8 -> v9

In oidc-spa v9 we've mainly removed the legacy API that have been maintain for a while for avoiding breaking changes while the newer APIs are stablilized. &#x20;

Overview of the changes:

* `oidc-spa/react` and `oidc-spa/mock/react` becomes `oidc-spa/react-spa`
* `oidc-spa/mock` has been moved to `oidc-spa/core-mock`
*



## Vite Plugin and Entrypoint

{% tabs %}
{% tab title="Vite Plugin" %}
{% code title="vite.config.ts" %}
```diff

  oidcSpa({
-   enableTokenExfiltrationDefense: true,
-   resourceServersAllowedHostnames: ["s3.amazon.com"],
+   browserRuntimeFreeze: { enabled: true },
+   tokenSubstitution: {
+       enabled: true,
+       trustedThirdPartyResourceServers: [ "s3.amazonaws.com" ]
+   }   
});
```
{% endcode %}

Or (if you have an older config):&#x20;

{% code title="vite.config.ts" %}
```diff
 oidcSpa({
-    freezeFetch: false,
-    freezeXMLHttpRequest: false,
-    freezeWebSocket: true,
-    freezePromise: true
+    browserRuntimeFreeze: { 
+        enabled: true,
+        exclude: ["fetch", "XMLHttpRequest" ]
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
-   resourceServersAllowedHostnames: ["s3.amazon.com"],
+   browserRuntimeFreeze: { enabled: true }
+   extraDefenseHook: () => {
+       enableTokenSubstitution({
+          trustedThirdPartyResourceServers: [ "s3.amazonaws.com" ]
+       });
+   }  
 });
```
{% endcode %}

Or (if you migrate from an older config)

{% code title="src/main.ts" %}
```diff
 oidcEarlyInit({
-    freezeFetch: false,
-    freezeXMLHttpRequest: false,
-    freezeWebSocket: true,
-    freezePromise: true
+    browserRuntimeFreeze: { 
+        enabled: true,
+        exclude: ["fetch", "XMLHttpRequest" ]
+    }  
});
```
{% endcode %}
{% endtab %}
{% endtabs %}
