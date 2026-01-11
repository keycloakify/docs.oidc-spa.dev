---
description: Polyfilling keycloak-js with oidc-spa
icon: arrow-up-to-dotted-line
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/oygeayjvIPxroUcp3jt4/resources/migrating-from-keycloak-js
---

# Migrating from Keycloak-js

If you're using [keycloak-js](https://www.npmjs.com/package/keycloak-js) in an existing codebase, you can migrate to `oidc-spa` without a painful rewrite.\
`oidc-spa` ships a `keycloak-js` polyfill. It’s a literal drop-in replacement.

### Why switch?

#### Security

* [Enabling DPoP](../security-features/dpop.md): Keycloak, [starting with 26.4](https://www.keycloak.org/2025/10/dpop-support-26-4), officially supports DPoP. `keycloak-js` doesn’t.
* [Browser Runtime Freeze](../security-features/browser-runtime-freeze.md)
* (Optional) [Token Substitution](../security-features/token-substitution.md)
* Static valid redirect URIs: `keycloak-js` forces you to allow wildcard redirects like `https://dashboard.my-company.com/*`. This is [a known attack vector](https://securityblog.omegapoint.se/en/writeup-keycloak-cve-2023-6927/). With `oidc-spa`, the only valid redirect URI is your app’s origin, for example `https://dashboard.my-company.com/`.

#### UX

* [Auto Logout](../features/auto-logout.md): optional “You will be logged out in 30…29…” overlay. No more “submit → redirect to login” because the Keycloak session expired.
* Login/Logout propagation across tabs.
* Much faster and relyable SSO, especially in non ideal condition (iframe blocked / Keycloak not on same site, slow network...)

### Update dependency

Replace `keycloak-js` with `oidc-spa` in your `package.json`.

{% code title="package.json" %}
```diff
 {
     dependencies: {
-        "keycloak-js": "...",
+        "oidc-spa": "..."
     }
 }
```
{% endcode %}

### Update your codebase

```diff
-import Keycloak from "keycloak-js";
+import { Keycloak } from "oidc-spa/keycloak-js";

 // ...

 await keycloak.init({
     onLoad: 'check-sso',
-    silentCheckSsoRedirectUri: `${location.origin}/silent-check-sso.html`,
     // ...
 });
```

Delete **public/silent-check-sso.html**.

### (OPTIONAL) Fix your Valid Redirect URIs

Log in to the Keycloak Admin Console. Open your client configuration.

```diff
Valid Redirect URIs:
 http://localhost*
-https://dashboard.my-company.com/*
-https://dashboard.my-company.com/silent-check-sso.html
+https://dashboard.my-company.com/
```

### Enable Security Features

If you're moving to `oidc-spa`, you likely want to [enable DPoP and other security features](../security-features/overview.md).

Pick the setup option that best fits your project:

{% tabs %}
{% tab title="Vite Plugin" %}
If you're in a Vite project, the recommended approach is to use `oidc-spa`’s Vite plugin.

<pre class="language-typescript" data-title="vite.config.ts"><code class="lang-typescript">import { defineConfig } from "vite";
<strong>import { oidcSpa } from "oidc-spa/vite-plugin";
</strong>
export default defineConfig({
    plugins: [
        // ...
<strong>        oidcSpa({
</strong><strong>            // See: https://docs.oidc-spa.dev/v/v10/security-features/browser-runtime-freeze
</strong><strong>            browserRuntimeFreeze: {
</strong><strong>                enabled: true
</strong><strong>                //exclude: [ "fetch", "XMLHttpRequest"]
</strong><strong>            },
</strong><strong>            // See: https://docs.oidc-spa.dev/v/v10/security-features/dpop
</strong><strong>            DPoP: {
</strong><strong>                enabled: true,
</strong><strong>                mode: "auto"
</strong><strong>            }
</strong><strong>        })
</strong>    ]
});
</code></pre>
{% endtab %}

{% tab title="Manual - Recommended" %}
Pick this approach if:

* You're not in a Vite project and
* Your app has a single client entrypoint.

***

Let's assume your app entrypoint is `src/main.ts`.

First, rename it to `src/main.lazy.ts`.

```bash
mv src/main.ts src/main.lazy.ts
```

Then create a new `src/main.ts` file:

{% code title="src/main.ts" %}
```typescript
import { oidcEarlyInit } from "oidc-spa/entrypoint";
import { browserRuntimeFreeze } from 'oidc-spa/browser-runtime-freeze';
import { DPoP } from 'oidc-spa/DPoP';

// Should run as early as possible.  
oidcEarlyInit({ 
    BASE_URL: "/" // The path where your app is hosted
                  // If applicable you should use `process.env.PUBLIC_URL`
                  // or `import.meta.env.BASE_URL`.
                  // This is not an option. There's only one good answer.
    securityDefenses: {
        // See: https://docs.oidc-spa.dev/v/v10/security-features/browser-runtime-freeze
        ...browserRuntimeFreeze({
            //exclude: [ "fetch", "XMLHttpRequest" ]
        }),
        // See: https://docs.oidc-spa.dev/v/v10/security-features/dpop
        ...DPoP({ mode: 'auto' })
    }
});
```
{% endcode %}
{% endtab %}

{% tab title="Manual - Easy" %}
If you’re not using Vite and you can’t edit your app’s entry file, run `oidcEarlyInit()` in the same module where you call `new Keycloak()`.

Note: this option [downgrades the security posture of your app](../security-features/overview.md#how-oidc-spa-achieves-this-in-a-nutshell) compared to the two other approaches. It can also conflict with some client-side routing libraries.

<pre class="language-typescript" data-title="src/oidc.ts"><code class="lang-typescript">import { Keycloak } from "oidc-spa/keycloak-js";
<strong>import { oidcEarlyInit } from "oidc-spa/entrypoint";
</strong><strong>import { browserRuntimeFreeze } from 'oidc-spa/browser-runtime-freeze';
</strong><strong>import { DPoP } from 'oidc-spa/DPoP';
</strong>
// Should run as early as possible.  
<strong>oidcEarlyInit({ 
</strong><strong>    BASE_URL: "/" // The path where your app is hosted
</strong><strong>                  // If applicable you should use `process.env.PUBLIC_URL`
</strong><strong>                  // or `import.meta.env.BASE_URL`.
</strong><strong>                  // This is not an option. There's only one good answer.
</strong><strong>    securityDefenses: {
</strong><strong>        // See: https://docs.oidc-spa.dev/v/v10/security-features/browser-runtime-freeze
</strong><strong>        ...browserRuntimeFreeze({
</strong><strong>            //exclude: [ "fetch", "XMLHttpRequest" ]
</strong><strong>        }),
</strong><strong>        // See: https://docs.oidc-spa.dev/v/v10/security-features/dpop
</strong><strong>        ...DPoP({ mode: 'auto' })
</strong><strong>    }
</strong><strong>});
</strong>
const keycloak = new Keycloak({ /* ... */ });
</code></pre>
{% endtab %}
{% endtabs %}

You can enable `keycloak.init({ enableLogging: true })` to see a console report for the security features.

### (OPTIONAL) Display a Warning Before Auto Logout

`oidc-spa` implements auto logout by respecting the idle session lifetime you configured in Keycloak.

To warn the user when they are about to be logged out due to inactivity, you can show an overlay like:\
“Are you still here? Your session will expire in 30…29…”

Get the underlying `oidc-spa` core object like this:

```typescript
import { Keycloak } from "oidc-spa/keycloak-js";

const keycloak = new Keycloak({ ... });

await keycloak.init({ ... });

// Can call this only after keycloak.init() has resolved.
const oidc = keycloak.getOidc();
```

Then implement the overlay as described here: [Displaying a Warning Before Auto Logout](../features/auto-logout.md#displaying-a-warning-before-auto-logout).
