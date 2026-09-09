---
description: Why Doesn't oidc-spa Require a Client Secret?
icon: question
---

# Why No Client Secret?

You might be wondering why `oidc-spa` doesn’t require a client secret and how it securely authenticates users without a backend handling the token exchange with the OIDC provider.

The key lies in the difference between **Authorization Code Flow**, which requires a client secret, and **Authorization Code Flow with PKCE**, which does not.

***

## Understanding the Two Variants of the Authorization Code Flow

**OIDC defines two common variants of the Authorization Code Flow:**

* **Authorization Code Flow:**\
  Requires a backend to perform the token exchange, since a _client secret_ is needed to securely obtain tokens.\
  In this model, the **server** acts as the OIDC client application.\
  Frameworks like **NextAuth** follow this approach.\
  The resulting access token is mostly incidental, it's used only if we need to call third party APIs.
* **Authorization Code Flow with PKCE:**\
  Adds an additional verification step that removes the need for a client secret, enabling secure token exchange directly from the **browser** (public client)**.**\
  This is the flow implemented by **oidc-spa**.\
  Here, the **frontend** itself is the OIDC client application, and the access token is used as a key to make authenticated requests to a backend that otherwise has no built-in knowledge of authentication.

So the **Authorization Code Flow** is intended for **server side OIDC** and **Authorization Code Flow with PKCE** is intended for **Browser side OIDC**.

What's a bit confusing is that some server side OIDC solution will also implement PKCE. They do that only as an extra layer of security in case the client secret get's leaked.

### 1. Authorization Code Flow (without PKCE)

This flow is typically implemented as follows:

1. The frontend initiates authentication but does **not** exchange the authorization code directly.
2. Instead, the backend receives the authorization code and uses a **client secret** to exchange it for tokens.
3. The backend stores the **access and refresh tokens** in a database.
4. The backend issues an **HttpOnly session cookie** to the frontend.
5. The frontend communicates with the backend, which retrieves and attaches access tokens to API requests using the session identifier stored in the cookie.

***

### 2. Authorization Code Flow + PKCE (Used by `oidc-spa`)

The standard Authorization Code Flow alone is insufficient for securely exchanging tokens directly from the browser, as it requires a client secret.\
Since anything embedded in frontend code is not truly secret, a different approach is needed.

This is where **PKCE** (Proof Key for Code Exchange) comes in. Here’s how it works with `oidc-spa`:

1. When the user needs to log in, either by clicking a "Login" button or by navigating to a protected part of the app, `oidc-spa` redirects them to the **OIDC provider's login page** (e.g., Keycloak, Auth0).
2. After successful authentication, the **OIDC provider establishes a session**, sets an `HttpOnly` session cookie in the browser, and redirects the user back to the app with an authorization code.
3. `oidc-spa` exchanges this code for tokens, completing a cryptographic challenge that eliminates the need for a client secret.
4. **Tokens remain in memory only,** `oidc-spa` does not store them in localStorage, sessionStorage, or a backend database.
5. When the user refreshes the page or revisits the app, `oidc-spa` **restores the session** by querying the OIDC provider in the background.
6. The OIDC provider uses the session cookie to determine whether the session is still valid. If it is, fresh tokens are issued **without requiring the user to log in again**.
7. If the session has expired, the user is redirected to the login page when accessing a protected area.

> **Note:** You might be concerned about the use of cookies, but here we are referring to **session cookies**, which do not require GDPR consent and are always enabled in all browsers.\
> A user who disables all cookies would not be able to use any website requiring authentication.\
> Session cookies should not be confused with **tracking cookies** or [**third-party cookies**](third-party-cookies-and-session-restoration.md).

***

## How `oidc-spa` Mitigates the Risks of Token Exposure

{% content-ref url="../security-features/overview.md" %}
[overview.md](../security-features/overview.md)
{% endcontent-ref %}

Read more:

{% embed url="https://auth0.com/docs/get-started/authentication-and-authorization-flow/authorization-code-flow-with-pkce" %}
