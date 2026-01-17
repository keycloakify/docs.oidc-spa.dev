---
icon: up
---

# v8 -> v10

This is the direct migration path from `oidc-spa` **v8** to **v10**.

It’s basically `v8 → v9` + `v9 → v10`, but written as a single flow.

### 1) Renames (v8 → v9)

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

- const { unusbscribe } = oidc.subscribeToTokensChange(...);
+ const { unsubscribeFromTokensChange } = oidc.subscribeToTokensChange(...);

createOidc({
-   noIframe: true
+   sessionRestorationMethod: "full page redirect"
});
```

### 2) `homeUrl` removed (v8 → v9)

```diff
createOidc({ // createMockOidc({
-   homeUrl: import.meta.env.BASE_URL,
});
```

{% tabs %}
{% tab title="Vite Plugin" %}
That’s it.
{% endtab %}

{% tab title="Manual" %}
The base url is now provided to `oidcEarlyInit`:

```diff
oidcEarlyInit({
+   BASE_URL: import.meta.env.BASE_URL
});
```
{% endtab %}
{% endtabs %}

### 3) Security defenses config (v8 → v10)

In v9, security defenses were reworked. In v10, `DPoP` became global, and token-substitution options were renamed.

#### Vite plugin

{% tabs %}
{% tab title="From v8 config (enableTokenExfiltrationDefense / resourceServersAllowedHostnames)" %}
{% code title="vite.config.ts" %}
```diff
oidcSpa({
-  enableTokenExfiltrationDefense: true,
-  resourceServersAllowedHostnames: ["s3.amazonaws.com"],
+  browserRuntimeFreeze: { enabled: true },
+  tokenSubstitution: {
+    enabled: true,
+    trustedExternalResourceServers: [
+      // Add your own wildcard if you want to trust "same site" subdomains.
+      "*.my-domain.com",
+      "s3.amazonaws.com"
+    ]
+  },
+  DPoP: { mode: "auto" /* or "enforced" */ }
});
```
{% endcode %}
{% endtab %}

{% tab title="From v8 config (older freeze* flags)" %}
{% code title="vite.config.ts" %}
```diff
oidcSpa({
-  freezeFetch: false,
-  freezeXMLHttpRequest: false,
-  freezeWebSocket: true,
-  freezePromise: true
+  browserRuntimeFreeze: {
+    enabled: true,
+    exclude: ["fetch", "XMLHttpRequest"]
+  },
+  tokenSubstitution: {
+    enabled: true,
+    trustedExternalResourceServers: [
+      "*.my-domain.com",
+      "s3.amazonaws.com"
+    ]
+  },
+  DPoP: { mode: "auto" /* or "enforced" */ }
});
```
{% endcode %}
{% endtab %}
{% endtabs %}

#### Manual (no Vite plugin)

{% tabs %}
{% tab title="From v8 config (enableTokenExfiltrationDefense / resourceServersAllowedHostnames)" %}
{% code title="src/main.ts" %}
```diff
import { oidcEarlyInit } from "oidc-spa/entrypoint";
-import { enableTokenSubstitution } from "oidc-spa/token-substitution";
+import { browserRuntimeFreeze } from "oidc-spa/browser-runtime-freeze";
+import { DPoP } from "oidc-spa/DPoP";
+import { tokenSubstitution } from "oidc-spa/token-substitution";

const { shouldLoadApp } = oidcEarlyInit({
-  enableTokenExfiltrationDefense: true,
-  resourceServersAllowedHostnames: ["s3.amazonaws.com"],
+  securityDefenses: {
+    ...browserRuntimeFreeze({
+      // exclude: ["fetch", "XMLHttpRequest", "Promise"]
+    }),
+    ...DPoP({ mode: "auto" /* or "enforced" */ }),
+    ...tokenSubstitution({
+      trustedExternalResourceServers: [
+        "s3.amazonaws.com",
+        "*.my-domain.com"
+      ]
+    })
+  }
});
```
{% endcode %}
{% endtab %}

{% tab title="From v8 config (older freeze* flags)" %}
{% code title="src/main.ts" %}
```diff
import { oidcEarlyInit } from "oidc-spa/entrypoint";
+import { browserRuntimeFreeze } from "oidc-spa/browser-runtime-freeze";
+import { DPoP } from "oidc-spa/DPoP";
+import { tokenSubstitution } from "oidc-spa/token-substitution";

oidcEarlyInit({
-  freezeFetch: false,
-  freezeXMLHttpRequest: false,
-  freezeWebSocket: true,
-  freezePromise: true
+  securityDefenses: {
+    ...browserRuntimeFreeze({
+      exclude: ["fetch", "XMLHttpRequest"]
+    }),
+    ...DPoP({ mode: "auto" /* or "enforced" */ }),
+    ...tokenSubstitution({
+      trustedExternalResourceServers: [
+        "s3.amazonaws.com",
+        "*.my-domain.com"
+      ]
+    })
+  }
});
```
{% endcode %}
{% endtab %}
{% endtabs %}

#### Takeaways (v9 → v10)

* `trustedThirdPartyResourceServers` was renamed to `trustedExternalResourceServers`.
* Same-site wildcards (like `*.my-domain.com`) are **not** trusted by default anymore. Add them explicitly if you relied on that.
* DPoP is now configured globally via `DPoP` / `DPoP: { ... }`.

### 4) Remove per-client `dpop` option (v9 → v10)

If you previously had this:

```diff
createOidc({ // or bootstrapOidc({
  // ...
- dpop: "auto"
});
```

Remove it. DPoP is now configured globally (see above).

### 5) Update of `keycloakUtils.getAccountUrl()` API (v8 → v9)

The “back to app” URL now must be a valid redirect URI:

```diff
const accountUrl = keycloakUtils.getAccountUrl({
  clientId: oidc.clientId,
- backToAppFromAccountUrl: location.href,
+ validRedirectUri: oidc.validRedirectUri,
  locale: "en" // Optional
});
```

### 6) Removal of `oidc-spa/tools/parseKeycloakIssuerUri` (v8 → v9)

There are now Keycloak utilities:

```diff
-import { parseKeycloakIssuerUri } from "oidc-spa/tools/parseKeycloakIssuerUri";
-
-const issuerUri = oidc.params.issuerUri;
-const clientId = oidc.params.clientId;
-
-const keycloak = parseKeycloakIssuerUri(issuerUri);
-
-if (keycloak === undefined) {
-  console.log("Not keycloak");
-  return;
-}
-
-const { origin, realm, kcHttpRelativePath, adminConsoleUrl, getAccountUrl } = keycloak;
-
-const accountUrl = getAccountUrl({
-  thisAppDisplayName: clientId,
-  backToAppFromAccountUrl: location.href
-});
+import { createKeycloakUtils, isKeycloak } from "oidc-spa/keycloak";
+
+const issuerUri = oidc.issuerUri;
+const clientId = oidc.clientId;
+const validRedirectUri = oidc.validRedirectUri;
+
+if (!isKeycloak({ issuerUri })) {
+  console.log("Not keycloak");
+  return;
+}
+
+const keycloakUtils = createKeycloakUtils({ issuerUri });
+
+const { origin, realm, kcHttpRelativePath } = keycloakUtils.issuerUriParsed;
+
+const { adminConsoleUrl } = keycloakUtils;
+
+const accountUrl = keycloakUtils.getAccountUrl({
+  clientId,
+  validRedirectUri,
+  locale: "en" // Optional
+});
```

### 7) React entrypoint rename (breaking) (v8 → v9)

{% hint style="warning" %}
Breaking change. `oidc-spa/react` has been removed. Use `oidc-spa/react-spa`.
{% endhint %}

Use the new integration guide:

[React SPA integration guide](https://app.gitbook.com/s/oygeayjvIPxroUcp3jt4/integration-guides/example-setups)

### 8) Server entrypoint rename (breaking) (v8 → v9)

{% hint style="warning" %}
Breaking change. `oidc-spa/backend` has been removed. Use `oidc-spa/server`.
{% endhint %}

Use the new docs:

[Server integration guide](https://app.gitbook.com/s/oygeayjvIPxroUcp3jt4/integration-guides/backend-token-validation)

### 9) `crypto.subtle` polyfill (v8 → v9)

`oidc-spa` now auto-polyfills `crypto.subtle` when it’s missing.

If you previously added `webcrypto-liner-shim`, you can remove it.

### Need a hand?

If you hit a migration edge case, ask on Discord:

<a href="https://discord.gg/mJdYJSdcm4" class="button secondary">Discord invite</a>
