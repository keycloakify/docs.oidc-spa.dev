---
icon: sliders
---

# Other OIDC Provider

If you are using an OIDC Provider other than the ones for wich we have [a specific guide for](broken-reference), here are the general instruction on how to configure your OIDC Provider. &#x20;

## Creating the Client Application

* Create a **Public** OpenID Connect Client.&#x20;
  * OpenID Connect Client can be reffered to as **OIDC Client** or **OAuth Client**.
  * The technical term for Public OIDC Client is: **Authorization Code Flow + PKCE**.
  * If you are provided with the option, **disable client credentials,** you do not need to provide any client secret to oidc-sp&#x61;**.**
  * Some provider will simply ask you to select an application type and let you chose between Single Page Application (SPA), Web Application (Or Web Server App) and Mobile App. **Select SPA**.
  * You will either be asked to explicitely provide a Client ID or an ID will be automatically generated. This is the `clientId` parameter that oidc-spa require.
* Valid Redirect URIs: **https://my-app.com/** and **http://localhost:5173/**
  * The slash at the end is important
  * If your app is hosted on a sub path like /dashboard you would set **https://my-app.com/dashboard/** and **http://localhost:5173/dashboard/**
  * 5173 is the default port used by Vite dev server, adapt according to your setup.
* Valid post logout redirect: Same as the Valid Redirec URIs
* Web Origins: **https://my-app.com**, **http://localhost:5173**\


## How do I find the IssuerURI?

The issuer URI is unfortunately not always clearly comunicated.  \
It depend on the provider.  \
If you are provided with a Discovery URL looking like:&#x20;

`https://XXX/.well-known/openid-configuration`&#x20;

You issuerUri is https://XXX

If you suspect that a given URL could be your issuer URI but you're unsure, just append /.well-known/openid-configuration to it and paste the URL in a web browser. If you get a JSON in response, you have your issuer URI! &#x20;

## Scopes and/or Audience

It's somewhat common for OIDC provider to make the client (oidc-spa) explicitely a specific Scope or Audience to issue an JWT access token.  \
Unfortunately how to configure this is very different from one OIDC provider to the next.  \


For example Auth0 make you ["Create an API" and specify an audience](auth0.md#creating-an-api).

Microsoft Entra ID makes you ["register an Application" and specify a scope](microsoft-entra-id.md#configuring-entra-id-to-issue-a-jwt-access-token).
