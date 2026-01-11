---
description: Polifilling keycloak-js with oidc-spa
icon: arrow-up-to-dotted-line
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/oygeayjvIPxroUcp3jt4/resources/migrating-from-keycloak-js
---

# Migrating from Keycloak-js

If you're using [keycloak-js](https://www.npmjs.com/package/keycloak-js) in an existing codebase you can migrate to oidc-spa without having to go through a painfull migration process. oidc-spa exposes a keycloak-js polyfill that is a literal drop in replacement.&#x20;

Why should you make the move?

Drastically improving the security posture of your app

* [Enabling DPoP](../security-features/dpop.md): Keycloak, [starting with version 26.4](https://www.keycloak.org/2025/10/dpop-support-26-4), officially support DPoP. However Keycloak-js doesn't.
* [Brwoser Runtime Freeze](../security-features/browser-runtime-freeze.md)
* (Optionally) [Token Substitution](../features/tokens-renewal.md)
* Non dynamic Valid Redirect URI: Keycloak-js forces you do have redirect uri like https://dashboard.my-company.com/\* wich is [a know attack vector](https://securityblog.omegapoint.se/en/writeup-keycloak-cve-2023-6927/). With oidc-spa the only valid redirect uri is the home of your app (https://dashboard.my-company.com/)

UX:

* [Auto Logout](../features/auto-logout.md) ("You will be logged out in 30...29..." overlay), no more the user fill a form, click on "submit" and then get redirected to the login because the session has expired on the keycloak side.
* Login/Logout propagation across tabs.
* Much smoother and faster session restoration when third party cookies are blocked.

{% stepper %}
{% step %}
### Update dependency

Replave keycloak-js by oidc-spa in your package.json

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
{% endstep %}

{% step %}
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
{% endstep %}

{% step %}
### (OPTIONAL) Fix your Valid Redirect URI

Log in to the Keycloak Admin Console, navigate in your client configuration.

```diff
Valid Redirect URI:
 http://localhost*
-https://dashboard.my-company.com/*
-https://dashboard.my-company.com/silent-check-sso.html
+https://dashboard.my-company.com/
```
{% endstep %}

{% step %}
### Enable Security Features

If you're moving to oidc-spa, you certainly want to [enable DPoP and other security feature](../security-features/overview.md).

Pick one of three setup option that best fit your setup:

{% tabs %}
{% tab title="Vite Plugin" %}
If you're in a Vite project, the recomended approach is to use oidc-spa's Vite plugin.

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

Note however that implementing this option [dowgrade the security posture of your app](../security-features/overview.md#how-oidc-spa-achieves-this-in-a-nutshell) compared to the two other approaches and, in some instances, might conflict with your client side routing library.

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

You can enabled `keycloak.init({ enableLogging: true })` to have a report in the console of the status of the security features. &#x20;
{% endstep %}

{% step %}
### (OPTIONAL) Displaying A Warning Before Auto Logout

With oidc-spa you get automatic auto logout, meaning that oidc-spa will respect the idle session Lifetime that you've define on the Keycloak side. &#x20;

To warn the user when they ar about to be auto logged out du to inactivity, you might want to implement an overlay like "Are you still here? Your session will expires in 30...29..."

To do that you can get the underlying oidc-spa core object with:

```typescript
import { Keycloak } from "oidc-spa/keycloak-js";

const keycloak = new Keycloak({ ... });

await keycloak.init({ ... });

// Can call this only after keycloak.init() has resolved.
const oidc = keycloak.getOidc();
```

Then with the oidc object, you can implement the overlay as described here: [#displaying-a-warning-before-auto-logout](../features/auto-logout.md#displaying-a-warning-before-auto-logout "mention")
{% endstep %}
{% endstepper %}
