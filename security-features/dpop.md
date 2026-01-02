---
description: OAuth 2.0 Demonstrating Proof-of-Possession
icon: receipt
---

# DPoP

[Demonstrating Proof-of-Possession (DPoP)](https://auth0.com/docs/secure/sender-constraining/demonstrating-proof-of-possession-dpop) is a protocol-level security mechanism defined in [**RFC 9449**](https://datatracker.ietf.org/doc/html/rfc9449).

It ensures that **an access token alone is no longer sufficient** to access a resource server.\
Instead, each request must also include a cryptographic proof showing possession of a private key held by the client.

As a result, access tokens become **much less sensitive**:\
if a token leaks, it cannot be replayed from another device or context without the corresponding private key.

DPoP [is supported by **Keycloak**](https://www.keycloak.org/2025/10/dpop-support-26-4) and an increasing number of other identity providers and resource server stacks.

***

## Enabling DPoP

oidc-spa exposes a single configuration option to control DPoP behavior:

* **`"auto"`**: enable DPoP only if supported by the authorization server, otherwise fall back to classic Bearer tokens (Recommended)
* **`"disabled"`**: never use DPoP (default)
* **`"enabled"`**: require DPoP support; oidc-spa will refuse to start if the authorization server does not support it. [See support history in Keycloak](https://www.keycloak.org/2025/10/dpop-support-26-4).

DPoP defaults to `"disabled"` because many resource servers still can’t validate DPoP-bound tokens.\
This keeps things working out of the box with older backends, until DPoP support becomes a baseline expectation for resource servers.

{% include "../.gitbook/includes/enable-security-defense.md" %}

***

## What does enabling DPoP require?

Enabling DPoP in oidc-spa does **not** require changes elsewhere in your stack:

* **Identity Provider (**[**Keycloak**](#user-content-fn-1)[^1] **or other)**\
  No configuration change is required.\
  With `dpop: "auto"`, If the authorization server supports DPoP, oidc-spa will detect and use it.
* **Frontend codebase**\
  No changes are required.\
  Authenticated requests continue to use `Authorization: Bearer <access_token>` and are automatically upgraded at runtime.
* **Backend API / resource server**\
  No code changes are required.\
  Just make sure your backend stack can validate and decode **DPoP-bound** access tokens.\
  This is supported by **Spring Security**, and of course by [**oidc-spa/server**](../integration-guides/backend-token-validation/).\
  There is nothing to “enable” on the resource server side.\
  If your RS supports DPoP, correct OAuth 2.0 token validation will reject DPoP-bound tokens when the DPoP proof is missing or invalid.\
  If your RS does not support DPoP, calls will simply fail, so there is no false sense of security.

In other words, **this configuration option is the only change required to securely enable DPoP support across your stack**.

## How it works

When DPoP is enabled, oidc-spa automatically **upgrades authenticated HTTP requests** sent by your application.

You continue sending requests as usual:

{% code title="Request headers (written by your code)" %}
```
Authorization: Bearer <access_token>
```
{% endcode %}

At runtime, oidc-spa transparently transforms the request into:

{% code title="Request headers (sent over the wire)" %}
```
Authorization: DPoP <access_token>
DPoP:          <DPoP proof JWT>
```
{% endcode %}

In addition, oidc-spa automatically:

* tracks and reuses DPoP nonces issued by resource servers
* retries requests when a nonce is required

To achieve this transparently, oidc-spa installs **`fetch()` and `XMLHttpRequest` interceptors**:

* via the Vite plugin, or
* during the execution of `oidcEarlyInit()`

From your application’s point of view, **nothing changes**:\
you keep using `Authorization: Bearer <access_token>`, and oidc-spa handles DPoP internally.

[^1]: DPoP is officially supported starting with version 26.4.\
    It's available as a preview feature in older versions.
