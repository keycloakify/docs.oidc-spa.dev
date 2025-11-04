---
description: How oidc-spa mitigates the risks of token exposure
icon: shield-check
---

# XSS and Supply-Chain Attack Protection

`oidc-spa` implements several security measures to minimize the risk of token theft, even when malicious JavaScript runs in your frontend (XSS or supply-chain attacks). This is especially relevant today given frequent reports of large-scale NPM supply-chain incidents.

`oidc-spa` treats the browser JavaScript runtime as a **hostile environment**.

To make defenses possible, `oidc-spa` needs a **safe window**: a guaranteed opportunity to run code **before any other JavaScript** executes. We achieve that with a Vite plugin and with an `oidcEarlyInit()` helper for other environments. This guarantee is essential — simply adding an import at the top of your entrypoint is not sufficient because module evaluation order is not deterministic. Any solution that does not offer an early initialization mechanism cannot claim the same level of protection.

---

## Baseline: no token persistence

The first and most important measure is to **avoid persisting tokens** — not in `localStorage`, not in `sessionStorage`.

When the app reloads, we restore the user session by contacting the IdP, which can restore the session via HTTP-only cookies. This is a current best practice, and many SDKs already follow it — `oidc-spa` goes further.

---

## Security measures unique to `oidc-spa`

Below are the primary protections. The first is common best practice; the rest are enabled by the early safe window and are unique to `oidc-spa`:

* **No token persistence** — avoid storing tokens in `localStorage`/`sessionStorage` whenever possible.
* **Freeze attack surface for network APIs** — we freeze or lock down `fetch`, `XMLHttpRequest`, and `WebSocket` so malicious code cannot monkey-patch them and exfiltrate tokens attached to requests.
* **Protect silent sign-in responses** — messages exchanged with iframes are protected by asymmetric encryption: the child encrypts the response with a public key provided by the parent; only the parent (which holds the private key in memory) can decrypt it. This prevents iframe message interception attacks like the one demonstrated in security talks.
* **Secure front-channel response handling** — authorization responses returned in callback URL parameters are moved to memory during initialization and then **cleared from the URL**, reducing the chance of accidental leakage (e.g., via logs, referrers, or extensions).

---

## Limitations — read this carefully

These mitigations significantly raise the bar for attackers, but they are **not** a mathematical proof of absolute safety. Important limitations:

* **Developer errors still expose tokens.** If application code explicitly logs or exposes tokens (e.g. `console.log(accessToken)` or `accessToken.split(...)`), an attacker can still capture them. Freezing builtins is limited to a small set of runtime APIs; freezing *everything* would be too intrusive and break many legitimate libraries.
* **Compromised browser extensions.** `oidc-spa` cannot and will not protect against malicious browser extensions. Extensions can observe network traffic or the DOM and are a different threat model that typically affects the end-user’s environment rather than your app itself.
* **Head-injected scripts / CDN polyfills.** If you include third-party scripts directly in `<head>` (CDN polyfills, analytics, etc.) they can execute before `oidc-spa` and negate protections. Importing JS from unknown CDNs is already a recognized security risk — avoid it.
* **No-iframe + multiple clients ⇒ persistence.** If your IdP is treated as third-party by the browser and your app uses multiple OIDC clients, `oidc-spa` may be forced to persist tokens in `sessionStorage` to avoid redirect loops. In that case, configure deployments so the IdP is first party to your app when possible. See: *Talking to multiple APIs*.
* **Service Worker edge case.** A malicious or compromised Service Worker could be used as an attack vector in complex, timed scenarios. The attack is hard to pull off but technically possible; disabling Service Workers removes this risk entirely. We are researching mitigations.

---

## The threat model `oidc-spa` defends against

To evaluate the protections, it helps to understand the attacker capabilities we assume.

### Variable scoping in JavaScript

If an attacker runs arbitrary JS in the same page (via XSS or a poisoned dependency) they will first try storage APIs (`localStorage`, `sessionStorage`). Because `oidc-spa` avoids persistence, tokens won't be found there.

Could an attacker just read in-memory variables that hold tokens? **No — not arbitrarily.** In JavaScript you cannot access local variables from outside their lexical scope:

```js
{
  const accessToken = "<secret>";
}
// ReferenceError: accessToken is not defined
console.log(accessToken);
```

Variables are only accessible if a reference to them exists in a reachable object. So simply holding tokens in local scope helps — but tokens still "float" through the environment (requests, event handlers, cross-window messages) and can be intercepted at those moments unless protected.

### Monkey-patching: the real practical attack

A common attack is to override global network APIs so the attacker inspects outgoing requests. For example:

```js
// Malicious code
const fetch_real = window.fetch;

window.fetch = function fetch(url, options){
  const authHeaderValue = options?.headers?.Authorization;
  if (authHeaderValue) {
    doSomethingMalicious(authHeaderValue.slice("Bearer ".length));
  }
  return fetch_real(url, options);
};

function doSomethingMalicious(accessToken){
  // send token to attacker server
}
```

After this runs, every authenticated request will leak the token. That is why `oidc-spa` **locks** `fetch`, `XMLHttpRequest`, and `WebSocket` early — to prevent replacement by malicious code.

Example of freezing `fetch` (conceptual):

```js
const fetch_trusted = globalThis.fetch;

Object.freeze(fetch_trusted);

Object.defineProperty(globalThis, "fetch", {
  configurable: false,
  writable: false,
  enumerable: true,
  value: fetch_trusted
});
```

We apply equivalent measures for `XMLHttpRequest` (used by older libraries like Axios) and `WebSocket`.

### iframe message interception

Iframe-based silent sign-in can be attacked by intercepting `postMessage` communications. To prevent this, `oidc-spa`:

* Encrypts the authorization response using a public key delivered to the iframe.
* Keeps the private key in memory only in the parent.
* Verifies origins and binds the key to the initialization process so it cannot be trivially overridden.

This defeats attacks that sniff or tamper with cross-window messages. See the referenced security talk for the attack demonstration.

{% embed url="https://www.youtube.com/watch?v=MpPd0WnEG5s&t=1272s" %}

---

## Summary — what `oidc-spa` achieves

* Provides **practical, deployable** mitigations that make token exfiltration via XSS or malicious dependencies **much harder**.
* Requires a build-time or early-init integration (Vite plugin / `oidcEarlyInit`) to create the safe window — without it, protections are unreliable.
* Does **not** replace secure development practices: avoid logging tokens, avoid loading untrusted scripts in `<head>`, and understand your threat model.
* Cannot defend against compromised browser extensions or a fully compromised client device — those are different attacker classes.

If you want the strongest protection `oidc-spa` can provide:

1. Use the provided Vite plugin or call `oidcEarlyInit()` as early as possible.
2. Avoid third-party scripts that run before your bundle.
3. Don’t persist tokens unless strictly required (and understand the tradeoffs).
4. Disable Service Workers if your threat model requires it.

If you’d like, I can help you add a short checklist to your docs that developers can follow to maximize safety in production.
