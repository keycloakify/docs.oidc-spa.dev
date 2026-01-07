---
description: How oidc-spa mitigates the risks of token exposure
icon: lighthouse
metaLinks:
  alternates:
    - https://app.gitbook.com/s/oygeayjvIPxroUcp3jt4/security-features/overview
---

# Overview

oidc-spa implements a comprehensive, defense-in-depth strategy to protect against token exfiltration during a successful XSS or supply-chain attack.

## Enabling the defences

With oidc-spa, all current best practices are implemented out of the box:

* **No persistence**: tokens live in memory only. Sessions are restored by contacting the Authorization server on every app reload.
* PKCE is always required and can't be disabled.
* Single, non-dynamic [valid redirect URI](#user-content-fn-1)[^1]. (As opposed to keycloak-js, which requires configuring a redirect URI with a wildcard, like https://dashboard.my-app.com/\*)

In addition to these baseline defences, oidc-spa offers three **opt-in** defences that drastically improve the security profile of your application.

<table data-view="cards"><thead><tr><th data-type="content-ref"></th></tr></thead><tbody><tr><td><a href="browser-runtime-freeze.md">browser-runtime-freeze.md</a></td></tr><tr><td><a href="dpop.md">dpop.md</a></td></tr><tr><td><a href="token-substitution.md">token-substitution.md</a></td></tr></tbody></table>

## Understanding the Security Guarantees (and Their Limits)

The objective of those defences is to achieve, **in a purely client-side token exchange**, a level of token safety comparable to traditional backend-based authentication (session cookies).\
The concerns that those oidc-spa defences address are described in this talk:

{% embed url="https://youtu.be/MpPd0WnEG5s?si=ZwlZujfmYboSMlE-&t=779" %}

With oidc-spa's defences enabled, an attacker cannot read or request valid tokens.

⸻

### Supply-Chain Attacks

If an NPM dependency is compromised, the damage remains extremely limited:

* With [DPoP](dpop.md), if a token gets exfiltrated, it's harmless outside of the call site. With [Token Substitution](token-substitution.md), the tokens are, in theory, not exfiltrable.
* This blocks the most common and impactful class of supply-chain attacks
* Most real-world supply-chain malware is opportunistic, not targeted

An attacker could theoretically act on behalf of the user during the active compromise, but:

* This requires a targeted attack specifically against your build
* This is realistic only for massive, high-value open-source systems
* Even then, oidc-spa makes it very difficult

Why? Because unlike session-cookie auth, where any `fetch()` automatically includes credentials, here the attacker must obtain a reference to your `fetchWithAuth()` or `getOidc()` functions.

These functions usually live inside hashed static assets (example: `assets/KcAdminUi-BV3D797K.js`). The hash will likely differ between the moment the attacker crafts the exploit and the moment the compromised dependency lands in your build.

Additionally, oidc-spa makes a best-effort attempt to make discovery of the module graph harder.

Bottom line: For supply-chain attacks, oidc-spa arguably offers stronger protection than traditional session cookies.

⸻

### XSS Attacks

XSS remains dangerous. oidc-spa protects against token exfiltration, but an attacker who knows everything about your build can still manage to act on behalf of the user while the attack is going on.

They can import your `fetchWithAuth()` implementation (exposed somewhere in the hashed JS assets) and perform any action the current user is allowed to perform.

Note that apps that implement traditional, backend-driven, session-cookie auth are just as vulnerable to XSS. It's even easier for the attacker since they don't even have to find the `fetchWithAuth` reference in the module graph; they can call the API with a simple `fetch()`, and the session cookie will be automatically attached.

The good news is that XSS can be very effectively blocked with strict Content-Security-Policy (CSP). And you should absolutely enable one.

Here you can find an example of a canonical, very strict CSP that ensures that only code that you own can run in your app:

[#canonical-nginx-configuration](../resources/csp-configuration.md#canonical-nginx-configuration "mention")

⸻

### Compromised Browser Extensions

If a user installs a malicious browser extension, it can inspect outgoing network traffic and see the real tokens.

Here's where [DPoP](dpop.md) shines. It makes it so that the access token alone is not enough to mint new requests, and prevents outgoing captured requests from being replayed.

However, [DPoP](dpop.md) is not an absolute protection since a malicious browser extension could theoretically manage to execute some code before oidc-spa's early init had the chance to ensure runtime integrity. This would, however, be very hard to pull off in practice. oidc-spa will block classical attack vectors.

Bottom line: oidc-spa makes it much, much harder for a compromised browser extension to successfully mint and exfiltrate usable tokens than any other client-side OIDC implementation. And in any case, such an attack would only affect the user with the compromised extension.

⸻

## How oidc-spa Achieves This (In a Nutshell)

The entire strategy relies on the fact that, thanks to the Vite plugin or `oidcSpaEarlyInit`, oidc-spa [gets a guaranteed window of execution before any other JavaScript runs](#user-content-fn-2)[^2].

During that window, it can:

* Harden the environment by preventing monkey-patching of fetch, XHR, WebSocket, Promise, String, and other critical built-ins
* Safely extract the authorization response from the URL and store it in memory
* Register a message listener that cannot be unregistered, ensuring silent-signin integrity
* Enforce restrictions on service worker registration
* And with DPoP and/or Token Substitution you're guaranteed either that a leaked token is harmless ([DPoP](dpop.md)) or that a token cannot be leaked ([Token Substitution](token-substitution.md)).

[^1]: Also referred to as "oidc callback uri"

[^2]: ...Unless you've opted to call oidcEarlyInit() in the oidc.ts file.
