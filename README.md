---
icon: up
---

# v8 -> v9

This release mostly renames entrypoints and config options.

Some changes are **breaking** if you use React (`oidc-spa/react`) or the server package (`oidc-spa/backend`).

### Import path changes

These are pure renames. A simple search/replace is enough:

```diff
- import { ... } from "oidc-spa";
+ import { ... } from "oidc-spa/core";

- import { ... } from "oidc-spa/mock";
+ import { ... } from "oidc-spa/core-mock";

- import { ... } from "oidc-spa/tools/decodeJwt";
+ import { ... } from "oidc-spa/decode-jwt";

- oidc.params.issuerUri;
+ oidc.issuerUri;

- oidc.params.clientId;
+ oidc.clientId;

- oidc.params.validRedirectUri;
+ oidc.validRedirectUri;


```

### Vite Plugin and oidcEarlyInit Params changes

oidc-spa's security features have been reworked, see:

{% content-ref url="https://app.gitbook.com/s/oygeayjvIPxroUcp3jt4/security-features/overview" %}
[Overview](https://app.gitbook.com/s/oygeayjvIPxroUcp3jt4/security-features/overview)
{% endcontent-ref %}

This is the changes you need to apply to migrate your current config while keeping the same security profile: &#x20;

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

### Removal of `oidc-spa/tools/parseKeycloakIssuerUri`

There is now more comprehensive keycloak integration utils: [Keycloak Utils](https://app.gitbook.com/s/oygeayjvIPxroUcp3jt4/features/keycloak-utils "mention")

```diff
-import { parseKeycloakIssuerUri } from "oidc-spa/tools/parseKeycloakIssuerUri";

-const issuerUri = oidc.params.issuerUri;
-const clientId = oidc.params.clientId;

-const keycloak = parseKeycloakIssuerUri(issuerUri);

-if( keycloak === undefined ){
-    console.log("Not keycloak");
-    return;
-}

-const { origin, realm, kcHttpRelativePath, adminConsoleUrl, getAccountUrl } = keycloak;

-const accountUrl = getAccountUrl({
-    thisAppDisplayName: clientId,
-    backToAppFromAccountUrl: location.href
-});

+ import { createKeycloakUtils, isKeycloak } from "oidc-spa/keycloak";

+const issuerUri = oidc.issuerUri;
+const clientId = oidc.clientId;
+const validRedirectUri = oidc.validRedirectUri;

+if( !isKeycloak({ issuerUri }) ){
+    console.log("Not keycloak");
+    return;
+}

+const keycloakUtils = createKeycloakUtils({ issuerUri });

+const { origin, realm, kcHttpRelativePath } = keycloakUtils.issuerUriParsed;

+const { adminConsoleUrl } = keycloakUtils;

+const accountUrl = keycloakUtils.getAccountUrl({
+    clientId: oidc.clientId,
+    validRedirectUri: oidc.validRedirectUri,
+    locale: "en" // Optional
+});
```

### React entrypoint rename (breaking)

{% hint style="warning" %}
Breaking change. `oidc-spa/react` has been removed. Use `oidc-spa/react-spa`.
{% endhint %}

The overall approach is the same, but the API changed significantly.

Use the new integration guide:

[React SPA integration guide](https://app.gitbook.com/s/oygeayjvIPxroUcp3jt4/integration-guides/example-setups)

### Server entrypoint rename (breaking)

{% hint style="warning" %}
Breaking change. `oidc-spa/backend` has been removed. Use `oidc-spa/server`.
{% endhint %}

This was required to support DPoP. It also cleans up the API.

Use the new docs: [Server integration guide](https://app.gitbook.com/s/oygeayjvIPxroUcp3jt4/integration-guides/backend-token-validation)

### `crypto.subtle` polyfill

`oidc-spa` now auto-polyfills `crypto.subtle` when it’s missing (typically when not served over HTTPS). This has no bundle size impact.

If you previously added `webcrypto-liner-shim` as described [here](https://app.gitbook.com/s/UhNOMoIddws1XoAnT5Nn/resources/fixing-crypto.subtle-is-available-only-in-secure-contexts-https), you can remove it.

### Need a hand?

If you hit a migration edge case, ask on Discord:

<a href="https://discord.gg/mJdYJSdcm4" class="button secondary">Discord invite</a>
