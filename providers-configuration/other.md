---
icon: sliders
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/oygeayjvIPxroUcp3jt4/providers-configuration/other
---

# Other OIDC Provider

If you are using an OIDC provider other than the ones for which we have [a specific guide](https://github.com/keycloakify/docs.oidc-spa.dev/blob/v6/providers-configuration/broken-reference/README.md), follow these general instructions to configure your OIDC provider.

{% hint style="warning" %}
Some providers don’t support SPAs as true **public OIDC clients**.

Before you proceed, make sure your provider supports:

* **Authorization Code Flow + PKCE**
* **Public clients** (no client secret; no client-credentials flow). If it lets you declare application type Single Page Application (SPA) you're good. &#x20;
* Configuring **redirect URIs** (login + post-logout)
* Configuring **web origins / CORS** for your app’s origin

Direct integration with “social login” providers (Google, GitHub, Facebook, etc.) is **not supported**. Use an identity platform like [Auth0](auth0.md) or [Microsoft Entra ID](microsoft-entra-id.md) to broker social logins.
{% endhint %}

## Creating the Client Application

* Create a **Public** OpenID Connect client.
  * OpenID Connect clients may also be referred to as **OIDC clients** or **OAuth clients**.
  * When asked, **disable client credentials,** or check **Public Client: true**.
  * Some providers will ask you to select an application type and choose between Single Page Application (SPA), Web Application (or Web Server App), and Mobile App. **Select SPA**.
  * You may need to explicitly provide a Client ID, or it may be generated automatically. This is the `clientId` parameter required by oidc-spa.
* **Valid Redirect URIs**:\
  **https://my-app.com/** and **http://localhost:**[**5173**](#user-content-fn-1)[^1]**/**
  * The trailing slash (`/`) is important.
  * If your app is hosted on a subpath (e.g., `/dashboard`), set:\
    **https://my-app.com/dashboard/** and **http://localhost:5173/dashboard/**
  * Port `5173` is the default for the Vite dev server; adjust as needed for your setup.
* **Valid Post-Logout Redirect URIs**:\
  Use the same values as the **Valid Redirect URIs**.
* **Web Origins**:\
  **https://my-app.com**, **http://localhost:5173**

## How Do I Find the `issuerUri`?

The issuer URI is not always clearly documented, it depends on the provider.

If you are given a Discovery URL like:

```
https://XXX/.well-known/openid-configuration
```

Then your `issuerUri` is:

```
https://XXX
```

If you suspect a URL might be the issuer URI but are unsure, append `/.well-known/openid-configuration` to it and open it in a web browser. If it returns a JSON response, then you have found your issuer URI!

## Getting JWT Access Tokens Issued

Many providers issue **opaque** access tokens by default. Opaque tokens require **introspection** on every request. Currently oidc-spa/server does not support them and probably never will because validating them requires aditional configuration setp and unessesary network roundtrip.

### Check what you’re currently getting

Copy your access token and look at its shape:

* `xxx.yyy.zzz` → **JWT**
* Anything else → **opaque**

### How providers usually make you get a JWT

1. Create an **API / Resource Server** in the provider.
2. Request tokens **for that API** during login.

The last step is provider-specific. You will usually pass one of these:

* `audience` (common on Auth0)
* an API `scope` (common on Entra ID)

In `oidc-spa`, this is usually one of these patterns:

```typescript
createOidc({
  // ...
  // Provider-specific API targeting:
  extraQueryParams: {
    // audience: "https://my-api",
  },
  // Provider-specific permissions:
  // scope: "openid profile email api.read"
});
```

### Examples

* [Auth0: create an API + set an audience](auth0.md#creating-an-api)
* [Microsoft Entra ID: configure the API + request a scope](microsoft-entra-id.md#configuring-entra-id-to-issue-a-jwt-access-token)

### Validate it on the backend

Once you get a JWT, validate it with the provider’s JWKS. See: [Backend Token Validation](../integration-guides/backend-token-validation/).

[^1]: This is the default port that Vite dev server uses. Addapt to your setup to be able to run your app in localhost.
