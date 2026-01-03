---
icon: up
---

# v8 -> v9

## Renamed Exports

No breaking changes, you can just search and replace.

```diff
- import { ... } from "oidc-spa";
+ import { ... } from "oidc-spa/core";

- import { ... } from "oidc-spa/mock";
+ import { ... } from "oidc-spa/core-mock";

- import { ... } from "oidc-spa/tools/decodeJwt";
+ import { ... } from "oidc-spa/decode-jwt";
```

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
     // Add to `exclude` the APIs for which you had freeze: false 
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
     // Add to `exclude` the APIs for which you had freeze: false
+    browserRuntimeFreeze: { 
+        enabled: true,
+        exclude: ["fetch", "XMLHttpRequest" ]
+    }  
});
```
{% endcode %}
{% endtab %}
{% endtabs %}

## Crypto.subtle polyfill

oidc-spa will now automatically polyfill crypto.subtle if missing due to your app not being deployed over HTTPS. It will do so without impact on the bundle size. &#x20;

If you had implemented&#x20;

## Removal of the `oidc-spa/react` export

The legacy `oidc-spa/react` export has been removed in favor of `oidc-spa/react-spa`.

The philosophy is the same but the API has changed substentially. &#x20;

You can follow the new integration guide to see the difference:

{% content-ref url="https://app.gitbook.com/s/oygeayjvIPxroUcp3jt4/integration-guides/example-setups" %}
[Getting Started](https://app.gitbook.com/s/oygeayjvIPxroUcp3jt4/integration-guides/example-setups)
{% endcontent-ref %}

Since the userbase of this older API is relatively small. I won't redact a full migration guide. However if you're facing difficulty upgrading I commit to help you on discord and even do the migration with you over a Discord call.

This is something I routinely do and that I like doing, I won't be surprize if you act on this offer.

[Discord Invite](https://discord.gg/mJdYJSdcm4)

