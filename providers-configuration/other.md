---
icon: sliders
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/UhNOMoIddws1XoAnT5Nn/providers-configuration/other
---

# Other OIDC Provider

If you are using an OIDC provider other than the ones for which we have [a specific guide](https://github.com/keycloakify/docs.oidc-spa.dev/blob/v6/providers-configuration/broken-reference/README.md), follow these general instructions to configure your OIDC provider.

## Creating the Client Application

* Create a **Public** OpenID Connect client.
  * OpenID Connect clients may also be referred to as **OIDC clients** or **OAuth clients**.
  * The technical term for a public OIDC client is **Authorization Code Flow + PKCE**.
  * If provided with the option, **disable client credentials**—you do not need to provide a client secret to oidc-spa.
  * Some providers will ask you to select an application type and choose between Single Page Application (SPA), Web Application (or Web Server App), and Mobile App. **Select SPA**.
  * You may need to explicitly provide a Client ID, or it may be generated automatically. This is the `clientId` parameter required by oidc-spa.
* **Valid Redirect URIs**:\
  **https://my-app.com/** and **http://localhost:5173/**
  * The trailing slash (`/`) is important.
  * If your app is hosted on a subpath (e.g., `/dashboard`), set:\
    **https://my-app.com/dashboard/** and **http://localhost:5173/dashboard/**
  * Port `5173` is the default for the Vite dev server; adjust as needed for your setup.
* **Valid Post-Logout Redirect URIs**:\
  Use the same values as the **Valid Redirect URIs**.
* **Web Origins**:\
  **https://my-app.com**, **http://localhost:5173**

## How Do I Find the `issuerUri`?

The issuer URI is not always clearly documented—it depends on the provider.

If you are given a Discovery URL like:

```
https://XXX/.well-known/openid-configuration
```

Then your `issuerUri` is:

```
https://XXX
```

If you suspect a URL might be the issuer URI but are unsure, append `/.well-known/openid-configuration` to it and open it in a web browser. If it returns a JSON response, then you have found your issuer URI!

## Scopes and Audience

Some OIDC providers require the client (`oidc-spa`) to explicitly request a specific **scope** or **audience** to issue a JWT access token.\
Unfortunately, the configuration varies significantly between providers.

For example:

* **Auth0** requires you to ["Create an API" and specify an audience](auth0.md#creating-an-api).
* **Microsoft Entra ID** requires you to ["register an application" and specify a scope](microsoft-entra-id.md#configuring-entra-id-to-issue-a-jwt-access-token).
