---
description: Why Doesn't oidc-spa Require a Client Secret
icon: question
---

# Why No Client Secret?

When working with OpenID Connect (OIDC), many developers expect to provide a **Client Secret** as part of the authentication process. This often leads to questions like:

* **"How come I do not need to provide a Client Secret?"**
* **"I thought OIDC token exchange was always handled on the backend?"**

The answer lies in the difference between **Authorization Code Flow + PKCE** (used by `oidc-spa`) and **Authorization Code Flow + Client Secret** (which requires a backend).

***

### **Understanding the Two Variants of the Authorization Code Flow**

The OIDC standard defines two main ways to implement the **Authorization Code Flow**:

* **Authorization Code Flow + Client Secret:** A straightforward token exchange flow that can only be implemented on a **backend**, where a secret can be securely stored.
* **Authorization Code Flow + PKCE:** A token exchange flow that involves additional steps but does not require the client application to hold any secret.

#### **1. Authorization Code Flow + PKCE (Used by `oidc-spa`)**

How it works:

* When the user logs in, the **OIDC provider (Keycloak, Auth0, etc.) establishes a session** and sets an `HttpOnly` cookie in the browser.
* `oidc-spa` **does not store tokens in localStorage, sessionStorage, or a backend database**—instead, it keeps them in memory.
* When the user refreshes the page or revisits the app, `oidc-spa` **automatically restores the session** by querying the OIDC provider in the background.
* If the session is still valid, fresh tokens are issued **without requiring the user to log in again**.
* If the session has expired, the user will only be redirected to log in **when they navigate to a part of the app that requires authentication**.

🔹 **Security benefit:** The **OIDC provider itself acts as the session store**, so there’s **no need for persistent token storage or a backend session manager**. This keeps the implementation simple and secure while maintaining a seamless user experience.

***

#### **2. Authorization Code Flow + Client Secret (Requires a Backend-for-Frontend)**

How it works:

1. The frontend initiates authentication but does **not** exchange the authorization code.
2. Instead, the backend receives the authorization code and uses a **Client Secret** to exchange it for tokens.
3. The backend stores the **access and refresh tokens in a database**.
4. The backend issues an **HttpOnly session cookie** to the frontend.
5. The frontend makes API requests via the backend, which retrieves and attaches the access token.

🔹 **Security benefit:** Since tokens are stored on the backend and never exposed to JavaScript, this approach **mitigates XSS risks**.

***

### **Why `oidc-spa` Uses PKCE Instead of a Client Secret**

We believe that any widely adopted open standard, implemented by every major OIDC provider, is secure enough for production—and PKCE falls squarely into that category.

While the theoretical security benefits of the flow with Client Secret is real, they come at a significant operational cost. Setting up and maintaining a secure BFF introduces complexity, potential misconfigurations, and new attack surfaces (session hijacking, CSRF, database leaks, etc.).

When you compare that to PKCE, which eliminates these concerns while following best security practices, the trade-off becomes clear. If the backend implementing the auth flow is client secret is not perfectly implemented, it can actually weaken security rather than enhance it.

For us the conclusion is simple: PKCE is the best option. It provides strong security guarantees without the added risks and complexity of a backend session management system.
