---
description: How oidc-spa mitigates the risks of token exposure
icon: shield-check
---

# XSS and Supply Chain Attack Defense

### Overview

oidc-spa implements a comprehensive, defense-in-depth strategy to protect against token exfiltration attacks.&#x20;

And with the advanced exfiltration model enabled, the security guarantees of a frontend-based approach become _theoretically equivalent_ to backend-based token exchange.

I say “theoretically” because this equivalence ultimately depends on the correctness of oidc-spa’s own hardening implementation. I believe the design is sound and the implementation is robust, but unlike backend token exchange, where tokens are never exposed by construction, I cannot claim mathematical certainty.

The approach uses **placeholder substitution** as its foundation: tokens exist as inert placeholder strings by default and are only resolved to real values at the moment they're needed for authorized network requests. This is fully transarent to you the user, it does not change the mental model, you do not need to adapt your code to make it work. This only inverts the traditional security model, instead of trying to block all possible exfiltration channels, we make the accessible data inherently safe by default.

### Enabling the Defense (Opt-In)

The token exfiltration defense is **opt-in** and can be enabled in two ways:

**1. Via Vite Plugin (Recommended):**

```typescript
// vite.config.ts
import { oidcSpa } from "oidc-spa/vite-plugin";

export default {
  plugins: [
    oidcSpa({
      enableTokenExfiltrationDefense: true,
      // If you access resource servers on other domain, you must declare them.
      //resourceServersAllowedHostnames: ["vault.my-company.com", "s3.my-company.com"],
    })
  ]
};
```

**2. Manual Setup:**

```typescript
import { oidcSpaEarlyInit } from "oidc-spa/earlyInit";

oidcSpaEarlyInit({
  enableTokenExfiltrationDefense: true,
  //resourceServersAllowedHostnames: [...]
});
```

### How It Works

#### 1. Placeholder Substitution

When enabled, all tokens (access token, ID token, refresh token) are replaced with placeholder strings like `access_token_placeholder_<id>`. These placeholders are the only values your application code ever sees.

#### 2. Comprehensive API Patching

oidc-spa patches the following browser APIs to intercept outgoing requests and substitute placeholders with real tokens for authorized destinations:

* `fetch` and `Request` constructor
* `XMLHttpRequest`
* `WebSocket`
* `EventSource`
* `navigator.sendBeacon`

#### 3. Host Authorization

Every patched network API enforces host-based authorization. Requests to unauthorized hosts fail with clear error messages guiding you to update your configuration. The defense automatically allows:

* Development servers (localhost, 127.0.0.1, \[::1])
* Same-origin requests
* Hosts matching your configured `resourceServersAllowedHostnames`
* Optionally, parent domain authorization for multi-tenant setups

#### 4. Anti-Tampering Measures

oidc-spa aggressively hardens the JavaScript environment to prevent circumvention:

* Prototypes are frozen to prevent subsequent monkey-patching
* Critical global objects are protected
* The defense initialization runs **before any other JavaScript** via build-time injection (Vite plugin)
* Attempts to tamper with the defense trigger explicit errors

### Strict Requirements: All-or-Nothing

**The defense refuses to start if any required condition is not met.** This is by design, a security defense is only as strong as its weakest point. If the environment cannot be properly hardened, oidc-spa will throw an error rather than provide false security.

{% hint style="success" %}
Important note: Even with enableTokenExfiltrationDefense: false oidc-spa still implement every single current best secrity practices for OIDC on the client. \
You'll still get zero token persistance and PKCE. Your app will still pass any security audit.
{% endhint %}

#### Known Incompatibilities

Libraries that monkey-patch network APIs **before** your code runs are incompatible with this defense. Examples include:

* **`@microsoft/applicationinsights-react-js`**: Monkey-patches `fetch` for telemetry
* Angular's Zone.js
* **OpenTelemetry auto-instrumentation**: Patches fetch/XHR for tracing
* **Any library that wraps or modifies native network APIs**
* **Any library that would register Service Worker from arbitrary blob.**

When such libraries are detected during initialization, oidc-spa will refuse to start with an error explaining the conflict.

**Workaround:** Use alternative libraries or configure them to use their own instrumentation hooks rather than monkey-patching.

### Limitations and CSP Requirement

#### This Does NOT Substitute for Content Security Policy

**Critical:** Token exfiltration defense is **one layer** of protection, not a complete security solution. You must still implement proper Content Security Policy (CSP) headers to prevent XSS attacks in the first place.

#### Residual Attack Surface

Even with token exfiltration defense enabled, a successful XSS attack still allows an attacker to:

1. **Act on behalf of the user**: The attacker can dynamically import application chunks and obtain a reference to your OIDC client instance (e.g., `oidc` from your React context). With this reference, they can call legitimate methods like `oidc.getAccessToken()` and make authorized API calls on the user's behalf.
2. **Perform CSRF-like actions**: The attacker can invoke application functionality that makes authorized requests.

**What the defense DOES prevent:**

* ✅ Silent token exfiltration to attacker-controlled servers
* ✅ Token theft for use outside the victim's browser session
* ✅ Token exposure in third-party analytics, monitoring, or compromised dependencies
* ✅ Supply chain attack opprotunisticly trying to perfrom some action on behafe of the user. &#x20;

**What the defense does NOT prevent:**

* ❌ In-session abuse via dynamically loaded application code
* ❌ XSS attacks themselves
* ❌ Social engineering or phishing

Note: The attacks oidc-spa does not prevent against, BFF and server side auth doesn't prevent either. &#x20;
