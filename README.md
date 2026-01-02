---
icon: up
---

# v8 -> v9

The security defences has been reworked. See security features overview:

{% content-ref url="https://app.gitbook.com/s/UhNOMoIddws1XoAnT5Nn/security-features/overview" %}
[Overview](https://app.gitbook.com/s/UhNOMoIddws1XoAnT5Nn/security-features/overview)
{% endcontent-ref %}



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
