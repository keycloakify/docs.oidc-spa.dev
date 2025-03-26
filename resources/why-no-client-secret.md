---
description: Why Doesn't oidc-spa Require a Client Secret?
icon: question
---

# Why No Client Secret?

You might be wondering why `oidc-spa` doesn’t require a client secret and how it securely authenticates users without a backend handling the token exchange with the OIDC provider.

The key lies in the difference between **Authorization Code Flow**, which requires a client secret, and **Authorization Code Flow with PKCE**, which does not.

***

## Understanding the Two Variants of the Authorization Code Flow

OIDC defines two common implementations of the **Authorization Code Flow**:

* **Authorization Code Flow:** Requires a backend to handle the token exchange, as a client secret is needed to obtain tokens securely.
* **Authorization Code Flow + PKCE:** Introduces an additional verification step, removing the need for a client secret, allowing secure token exchange directly from the client application.

### 1. Authorization Code Flow (without PKCE)

This flow is typically implemented as follows:

1. The frontend initiates authentication but does **not** exchange the authorization code directly.
2. Instead, the backend receives the authorization code and uses a **client secret** to exchange it for tokens.
3. The backend stores the **access and refresh tokens** in a database.
4. The backend issues an **HttpOnly session cookie** to the frontend.
5. The frontend communicates with the backend, which retrieves and attaches access tokens to API requests using the session identifier stored in the cookie.

***

### 2. Authorization Code Flow + PKCE (Used by `oidc-spa`)

The standard Authorization Code Flow alone is insufficient for securely exchanging tokens directly from the frontend, as it requires a client secret.\
Since anything embedded in frontend code is not truly secret, a different approach is needed.

This is where **PKCE** (Proof Key for Code Exchange) comes in. Here’s how it works with `oidc-spa`:

1. When the user needs to log in—either by clicking a "Login" button or by navigating to a protected part of the app—`oidc-spa` redirects them to the **OIDC provider's login page** (e.g., Keycloak, Auth0).
2. After successful authentication, the **OIDC provider establishes a session**, sets an `HttpOnly` session cookie in the browser, and redirects the user back to the app with an authorization code.
3. `oidc-spa` exchanges this code for tokens, completing a cryptographic challenge that eliminates the need for a client secret.
4. **Tokens remain in memory only**—`oidc-spa` does not store them in localStorage, sessionStorage, or a backend database.
5. When the user refreshes the page or revisits the app, `oidc-spa` **restores the session** by querying the OIDC provider in the background.
6. The OIDC provider uses the session cookie to determine whether the session is still valid. If it is, fresh tokens are issued **without requiring the user to log in again**.
7. If the session has expired, the user is redirected to the login page when accessing a protected area.

> **Note:** You might be concerned about the use of cookies, but here we are referring to **session cookies**, which do not require GDPR consent and are always enabled in all browsers.\
> A user who disables all cookies would not be able to use any website requiring authentication.\
> Session cookies should not be confused with **tracking cookies** or [**third-party cookies**](end-of-third-party-cookies.md).

## Advantages and Trade-offs of Implementing Token Exchange on the Frontend

✅ **No persistent token storage** – There’s no need to store user tokens in a backend database. The OIDC provider itself acts as the session store, meaning you only need to focus on\
securely deploying your OIDC server (if self-hosting).

✅ **Fewer moving parts** – Everything happens between `oidc-spa` and the OIDC provider, reducing the chances of misconfiguration.\
Even if you're not entirely confident in your setup, as long as it works, you’ve implemented it correctly.

However, in this mode, tokens are exposed to the **JavaScript client code**, unlike when the token exchange is performed on the backend.\
This introduces a potential risk: **XSS attacks**, where malicious code running on your website could attempt to steal tokens.

***

## How `oidc-spa` Mitigates the Risks of Token Exposure

`oidc-spa` implements several security measures to minimize the risk of token theft, even in the event of an XSS attack (malicious JavaScript running in your frontend).

> **Note:** This is a work in progress—some of these measures are not yet fully implemented, but they are actively being developed.

To fully benefit from these protections, ensure `oidc-spa` is the first JavaScript code that runs on your website by following [this setup](broken-reference).

Security Measures:

* **No persistent token storage** – Tokens are never stored in `localStorage` or `sessionStorage`. Instead, they are kept in scoped variables that are inaccessible to the global scope.
* **Preventing `fetch` and** `XMLHttpRequest` **monkey patching** – APIs that caries access tokens are frozen to prevent malicious code from overriding it and capturing tokens.
* **(WIP) Securing silent sign-in responses** – Even if an attacker intercepts the authorization response from a silent sign-in performed in an iframe, it will be asymmetrically encrypted, making it unusable.
* **(WIP) Secure transfer of the authorization response after front-channel login** – The authorization response, which is temporarily stored in session storage during the redirect process, will be cleared and moved to memory before any other code runs.

## Opinionated Conclusion

PKCE is a widely adopted open standard, supported by all major OIDC providers, and provides strong security guarantees **without requiring a backend**.

While a backend-based token exchange is theoretically more secure, in practice, it introduces additional attack surfaces and operational complexity. Every extra moving part is a potential point of failure or misconfiguration, and securing a backend against threats like token leakage, improper session management, and server-side vulnerabilities is a non-trivial task.

With the security measures implemented in `oidc-spa`, **Authorization Code Flow + PKCE is not just the simpler approach—it is arguably the safer one in real-world scenarios.** By eliminating the backend entirely, it reduces the risk of misconfiguration and ensures that authentication security is handled directly by the OIDC provider, which is purpose-built for this task.



Read more:

{% embed url="https://auth0.com/docs/get-started/authentication-and-authorization-flow/authorization-code-flow-with-pkce" %}
