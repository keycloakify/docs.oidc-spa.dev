---
description: How oidc-spa mitigates the risks of token exposure
icon: lighthouse
---

# Overview

{% hint style="danger" %}
Still in construciton, comme back in a few hours/days.
{% endhint %}

oidc-spa implements a comprehensive, defense-in-depth strategy to protect against token exfiltration during a successful XSS or supply-chain attack.

## Enabling the defences

With oidc-spa, all best current best practicies are implemented out of the box: &#x20;

* **No persistance**: tokens lives in memory only \*[^1], session are restored by contacting the Authorization server every app reload. &#x20;
* [PKCE](../resources/why-no-client-secret.md#id-2.-authorization-code-flow--pkce-used-by-oidc-spa) is always required and can't be disabled.
* Single, non dynamic [valid redirect uri](#user-content-fn-2)[^2]. (By oposition with keycloak-js that have requires to configure a redirect uri with wildchard, like https://dashboard.my-app.com/\*)

In addition to theses basline oidc-spa offers three **opt-in** defences that drastically improve the security profile of your application.

<table data-view="cards"><thead><tr><th data-type="content-ref"></th></tr></thead><tbody><tr><td><a href="browser-runtime-freeze.md">browser-runtime-freeze.md</a></td></tr><tr><td><a href="dpop.md">dpop.md</a></td></tr><tr><td><a href="token-substitution.md">token-substitution.md</a></td></tr></tbody></table>

## Understanding the Security Guarantees (and Their Limits)

The objective of those deffense is to achieve, **in a purely client-side token exchange**, a level of token safety comparable to traditional backend-based authentication (session cookies).\
The concerns that thoses oidc-spa defences addresses are described in this talk: &#x20;

{% embed url="https://youtu.be/MpPd0WnEG5s?si=ZwlZujfmYboSMlE-&t=779" %}

With oidc-spa's defences enabled, an attacker cannot read or request valid tokens.

⸻

### Supply-Chain Attacks

If an NPM dependency is compromised, the damage remains extremely limited:

* With [DPoP](dpop.md), if a token get exfiltred it's harmless outside of the call site, with [Token Substitution](token-substitution.md), the token are, in theory, not exfiltrable.&#x20;
* This blocks the most common and impactful class of supply-chain attacks&#x20;
* Most real-world supply-chain malware is opportunistic, not targeted

An attacker could theoretically act on behalf of the user during the active compromise, but:&#x20;

* This requires a targeted attack specifically against your build&#x20;
* This is realistic only for massive, high-value open-source systems&#x20;
* Even then, oidc-spa makes it very difficult

Why? Because unlike session-cookie auth, where any `fetch()` automatically includes credentials, here the attacker must obtain a reference to your `fetchWithAuth()` or `getOidc()` functions.

These functions usually live inside hashed static assets (example: `assets/KcAdminUi-BV3D797K.js`). The hash will likely differ between the moment the attacker crafts the exploit and the moment the compromised dependency lands in your build.

Additionally, oidc-spa make best effort attempt to block the discovery of the module graph harder.

Bottom line: For opportunistic supply-chain attacks, oidc-spa arguably offers stronger protection than traditional session cookies.

⸻

### XSS Attacks

XSS remains dangerous. oidc-spa protects agaist token exfiltration, but an attacker who would know everything about your build can still manage to act on behafe of user while the attack is going on.

They can import your `fetchWithAuth()` implementation (exposed somwere ine the hashed js assets) and perform any action the current user is allowed to perform.

The good news is that XSS can be very effectively blocked with strict Content-Security-Policy (CSP). And you should absolutely enable one.

Here you can find an example of a cannonical very strict CSP that ensure that only codes that you own can run in your app: &#x20;

[#canonical-nginx-configuration](../resources/csp-configuration.md#canonical-nginx-configuration "mention")

⸻

### Compromised Browser Extensions

This is the one scenario where cookie-based auth has an advantage over oidc-spa's client side auth.

If a user installs a malicious browser extension, it can inspect outgoing network traffic and see the substituted tokens.

This affects only the user with the compromised extension, and it affects all SPAs using client-side auth, not just your app.

⸻

## How oidc-spa Achieves This (In a Nutshell)

The entire strategy relies on the fact that, thanks to the Vite plugin or `oidcSpaEarlyInit`, oidc-spa [gets a guaranteed window of execution before any other JavaScript runs](#user-content-fn-3)[^3].

During that window, it can:&#x20;

* Harden the environment by preventing monkey-patching of fetch, XHR, WebSocket, Promise, String, and other critical built-ins&#x20;
* Safely extract the authorization response from the URL and store it in memory&#x20;
* Register a message listener that cannot be unregistered, ensuring silent-signin integrity&#x20;
* Enforce restrictions on service worker registration&#x20;
* And with DPoP and/or Token Substitution you're guarenteed either that a leaked token is harmless ([DPoP](dpop.md)) or that token cannot be leaked ([Token Substitution](token-substitution.md)).

[^1]: Except if your app talk to multiple different resource server AND your Authorization server hosted off site. In that specific scenario oidc-spa might need to persist token in sessionStorage. More details in the "Talking to multiple APIs" page.

[^2]: Also refered to as "oidc callback uri"

[^3]: ...Unless you've opted for calling oidcEarlyInit() in the oidc.ts file.
