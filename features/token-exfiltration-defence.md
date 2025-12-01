---
description: How oidc-spa mitigates the risks of token exposure
icon: shield-check
---

# Token Exfiltration Defence

{% hint style="warning" %}
The API of enableTokenExfiltrationDefense is not stable yet.  \
We're still actively working on those defences. They will evolves in the comming weeks. &#x20;
{% endhint %}

oidc-spa implements a comprehensive, defense-in-depth strategy to protect against token exfiltration during a successful XSS or supply-chain attack.

The objective is to achieve, **in a purely client-side architecture**, a level of token safety comparable to traditional backend-based authentication (session cookies).\
The concerns raised in the talk below no longer apply when the exfiltration defence is enabled. And even without the advanced defence enabled, the demo where they manually request a token would be structurally impossible with oidc-spa, an attacker cannot request new cretentials. &#x20;

{% embed url="https://youtu.be/MpPd0WnEG5s?si=ZwlZujfmYboSMlE-&t=779" %}

With the defence enabled, an attacker cannot read or request valid tokens.\
This is the same property provided by backend session cookies.

## Enabling the Exfiltration Defence

{% hint style="warning" %}
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
{% endhint %}

Enabling the defence is simply a matter of flipping a switch:

{% tabs %}
{% tab title="Vite Plugin" %}
<pre class="language-typescript" data-title="vite.config.ts"><code class="lang-typescript">import { oidcSpa } from "oidc-spa/vite-plugin";

export default {
  plugins: [
    // ...
    oidcSpa({
<strong>      enableTokenExfiltrationDefense: true,
</strong><strong>      // If you send auted request to third party server
</strong><strong>      // (outside of your site) you must declare them.
</strong><strong>      //resourceServersAllowedHostnames: [ "s3.amazonaws.com" ]
</strong>    })
  ]
};
</code></pre>
{% endtab %}

{% tab title="Manual Setup" %}
<pre class="language-typescript" data-title="src/main.ts"><code class="lang-typescript">import { oidcSpaEarlyInit } from "oidc-spa/earlyInit";

oidcSpaEarlyInit({
<strong>    enableTokenExfiltrationDefense: true,
</strong>    // If you send auted request to third party server
    // (outside of your site) you must declare them.
    //resourceServersAllowedHostnames: [ "s3.amazonaws.com" ]
});
</code></pre>
{% endtab %}
{% endtabs %}

⸻

## Understanding the Security Guarantees (and Their Limits)

### Supply-Chain Attacks

If an NPM dependency is compromised, the damage remains extremely limited:

* The attacker cannot exfiltrate valid tokens&#x20;
* This blocks the most common and impactful class of supply-chain attacks • Most real-world supply-chain malware is opportunistic, not targeted

An attacker could theoretically act on behalf of the user during the active compromise, but:&#x20;

* This requires a targeted attack specifically against your build&#x20;
* This is realistic only for massive, high-value open-source systems&#x20;
* Even then, oidc-spa makes it very difficult

Why? Because unlike session-cookie auth, where any `fetch()` automatically includes credentials, here the attacker must obtain a reference to your `fetchWithAuth()` or `getOidc()` functions.

These functions usually live inside hashed static assets (example: `assets/KcAdminUi-BV3D797K.js`). The hash will likely differ between the moment the attacker crafts the exploit and the moment the compromised dependency lands in your build.

Additionally, oidc-spa blocks the discovery of the module graph\[^1].

Bottom line: For supply-chain attacks, oidc-spa offers stronger protection than traditional session cookies.

⸻

### XSS Attacks

XSS remains dangerous. oidc-spa protects agaist token exfiltration, but an attacker who would know everything about your build can still manage to act on behafe of user while the attack is going on.

They can import your `fetchWithAuth()` implementation (exposed somwere ine the hashed js assets) and perform any action the current user is allowed to perform

The good news is that XSS can be very effectively blocked with strict Content-Security-Policy (CSP). And you should absolutely enable one.

{% content-ref url="../resources/csp-configuration.md" %}
[csp-configuration.md](../resources/csp-configuration.md)
{% endcontent-ref %}

⸻

### Compromised Browser Extensions

This is the one scenario where cookie-based auth has an advantage over oidc-spa's client side auth.

If a user installs a malicious browser extension, it can inspect outgoing network traffic and see the substituted tokens.

This affects only the user with the compromised extension, and it affects all SPAs using client-side auth, not just your app.

⸻

## How oidc-spa Achieves This

The entire strategy relies on the fact that, thanks to the Vite plugin or `oidcSpaEarlyInit`, oidc-spa gets a guaranteed window of execution before any other JavaScript runs.

During that window, it can:&#x20;

* Harden the environment by preventing monkey-patching of fetch, XHR, WebSocket, Promise, String, and other critical built-ins&#x20;
* Safely extract the authorization response from the URL and store it in memory&#x20;
* Register a message listener that cannot be unregistered, ensuring silent-signin integrity • Enforce restrictions on service worker registration&#x20;
* And most importantly:

Tokens are never exposed to the application layer

The tokens your app sees are structurally valid JWTs, but the signature segment is replaced. Such tokens cannot be used to authenticate requests.

Before any request leaves the app (fetch, XHR, WebSocket, beacon), the real tokens are restored inside a hardened, sandboxed pre-network interceptor created during early init.

The only way to see the real token is to inspect network traffic.

These protections have zero impact on DX or performance. The only requirement is to avoid libraries that monkey-patch critical built-ins and know ahead of time which resources server outside of your site your app might want to send authed request to (like s3.amazon.com for example). &#x20;

***
