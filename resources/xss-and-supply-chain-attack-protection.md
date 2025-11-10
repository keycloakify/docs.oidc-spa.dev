---
description: How oidc-spa mitigates the risks of token exposure
icon: shield-check
---

# XSS and Supply Chain Attack Protection

`oidc-spa` implements several security measures to minimize the risk of token theft, even when malicious JavaScript runs in your frontend (XSS or supply-chain attacks). This is especially relevant today given frequent reports of large-scale NPM supply-chain incidents.

`oidc-spa` treats the browser JavaScript runtime as a **hostile environment**.

To make defenses possible, `oidc-spa` needs a guaranteed opportunity to run code **before any other JavaScript** executes. We achieve that with a Vite plugin and, in non Vite environnements with an `oidcEarlyInit()` helper. This guarantee is essential, simply adding an import at the top of your entrypoint is not sufficient because module evaluation order is not deterministic. Any solution that does not offer an early initialization mechanism compled with lazy loading of your app's code cannot claim the same level of protection.

***

## Advantage over server-centric authentication

In server-centric models like Auth.js or BetterAuth, it can feel reassuring to know that tokens are never directly exposed to JavaScript. However, it’s important to understand that if an attacker manages to inject and execute code within your app’s origin, those systems impose virtually no limits on what the attacker can do. Even without direct access to the tokens, they can perform any action on behalf of the user, since authentication is handled automatically via cookies.

With oidc-spa, the situation is fundamentally different. Even if an XSS attack occurs, the attacker cannot send authenticated requests to your server unless they explicitly attach a valid access token. The security mechanisms implemented in oidc-spa, the ones detailed in this document, are designed to ensure that obtaining or refreshing such tokens for code that do not explicitly import oidc-spa at build time is virtually impossible.

***

## Baseline: no token persistence

The first and most important measure is to **avoid persisting tokens,** not in `localStorage`, not in `sessionStorage`.

When the app reloads, we restore the user session by contacting the IdP, which can restore the session via HTTP-only cookies. This is a current best practice, and many SDKs already follow it.  \
However this security mesure implemented in isolation is next to irrelevent. There's trival attacks that can be implemented even when the token are not persisted.  \
oidc-spa goes much futher than this.  &#x20;

***

## Security measures unique to `oidc-spa`

* **Freeze attack surface for network APIs**: we freeze or lock down `fetch`, `XMLHttpRequest`, and `WebSocket` so malicious code cannot monkey-patch them and exfiltrate tokens attached to requests.
* **Protect silent sign-in responses**: messages exchanged with iframes are protected by asymmetric encryption: the child encrypts the response with a public key provided by the parent; only the parent (which holds the private key in memory) can decrypt it. This prevents iframe message interception attacks like [the one demonstrated in security talks](https://www.youtube.com/watch?v=MpPd0WnEG5s\&t=1272s).
* **Secure front-channel response handling**: authorization responses returned in the callback URL parameters are moved to memory during the safe initialization window and then **cleared from the URL**.

***

## Limitations, read this carefully

These mitigations significantly raise the bar for attackers, but they are **not** a mathematical proof of absolute safety. Important limitations:

* **React Router Framework:** React Router when used in **Framework mode** [patches the entrypoint in a way that make it impossible to implement an exclusive execution window](https://github.com/keycloakify/oidc-spa/issues/110#issuecomment-3499101635). So, in this specific setup, oidc-spa cannot protect you the same way it can in every other solution. For better security guarentee consider using [TanStack Router/Start](../integration-guides/tanstack-router-start/). Note however that, unless you move your auth to the backend, no other oidc client implementation will provide you better security guarenty than oidc-spa, even in the context of RR Framwork. &#x20;
* **Developer can still expose tokens.** If application code explicitly manipulate tokens with builtin utils, e.g. `console.log(accessToken)` or `accessToken.split(...)`) an attacker can still capture them. In those examples via monkey patching of **console.log** or **String.prototpye.split**. oidc-spa's freezing builtins is limited to the set of runtime APIs that are relevent in sanctioned usecases; freezing _everything_ would be too intrusive and break many legitimate libraries.
* **Compromised browser extensions.** `oidc-spa` cannot protect against malicious browser extensions. Extensions can observe network traffic. This is an advantage that remainse for backen driven token exchange. **That being said**, you will never and could never be blamed for a user's compromised environement. This scenario would only affect a specific set of user that have that extention installed and every SPAs they visit would leak their token not just yours.
* **Head-injected scripts / CDN polyfills.** If you include third-party scripts directly in `<head>` (CDN polyfills, analytics, etc.) they can execute before `oidc-spa` and if they are compromized, can bypass oidc-spa protection. [Importing JS from CDNs is already a recognized security risk, avoid it](https://youtu.be/2xLawmYa_KY?si=sLDSN4iKLNLuOFAY\&t=871).
* **No-iframe + multiple clients ⇒ persistence.** If your IdP is treated as third-party by the browser and your app uses multiple OIDC clients, `oidc-spa` will persist tokens in `sessionStorage` to avoid redirect loops. If you're talking to multiple OAuth2 API (not only your backend), make sure to configure deployments so the IdP is first party to your app when possible. See: [_Talking to multiple APIs_](../talking-to-multiple-apis-with-different-access-tokens.md).
* **Service Worker edge case.** A malicious or compromised Service Worker could be used as an attack vector in complex, timed scenarios. The attack is hard to pull off but technically possible; disabling Service Workers removes this risk entirely. **We are researching mitigations**.

Important to note: Even if you where to be in the absolute worst case scenario for all those points. oidc-spa would still provide better security guarentee than any other OIDC client implementation (as of date). &#x20;

***

## The threat model `oidc-spa` defends against

To evaluate the protections, it helps to understand the attacker capabilities we assume.

### Variable scoping in JavaScript

If an attacker runs arbitrary JS in the same page (via XSS or a poisoned dependency) they will first try storage APIs (`localStorage`, `sessionStorage`). Because `oidc-spa` avoids persistence, tokens won't be found there.

Could an attacker just read in-memory variables that hold tokens? **No, not arbitrarily.** In JavaScript you cannot access local variables from outside their lexical scope:

```js
{
  const accessToken = "<secret>";
}
// ReferenceError: accessToken is not defined
console.log(accessToken);
```

Variables are only accessible if a reference to them exists in a reachable object. So simply holding tokens in local scope helps, but tokens still "float" through the environment (requests, event handlers, cross-window messages) and can be intercepted at those moments unless protected. This is why no persistence alone is not enough and oidc-spa goes futher. &#x20;

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
  // send token to the attacker's server
}
```

After this runs, every authenticated request will leak the token. That is why `oidc-spa` **locks** `fetch`, `XMLHttpRequest`, and `WebSocket` early, to prevent replacement by malicious code.

Example of freezing `fetch`

```js
Object.defineProperty(window, "fetch", {
  configurable: false,
  writable: false,
  enumerable: true,
  value: fetch
});

// This will fail
window.fetch = ()=> {};

// Still the original, unalterated, fetch.
fetch();
```

We apply equivalent measures for `XMLHttpRequest` (used by older libraries like Axios) and `WebSocket`.

### iframe message interception

Iframe-based silent sign-in can be attacked by intercepting `postMessage` communications. To prevent this, `oidc-spa`:

* Encrypts the authorization response using a public key delivered to the iframe.
* Keeps the private key in memory only in the parent.
* Verifies origins and binds the key to the initialization process so it cannot be trivially overridden.
* Ensure the public key used to encrypt the message has not been overwriten by another process before decoding the response in the parent. &#x20;
* We ensure that the child process that post the auth server response is running entirely in the safe window where no other code has run yet. &#x20;

This defeats attacks that sniff or tamper with cross-window messages as the one demonstrated in this talk:

{% embed url="https://www.youtube.com/watch?v=MpPd0WnEG5s&t=803s" %}

Everything explained and demonstrated in this talk is true and relevant. &#x20;

We just disagree with the conclustion that the ony solution is to move auth to the backend. We can mitigate thoses theat with client engeenering.&#x20;

***
