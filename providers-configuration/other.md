---
icon: sliders
---

# Other OIDC Provider

If you are using an OIDC Provider other than the ones for wich we have [a specific guide for](broken-reference), here are the general instruction on how to configure your OIDC Provider. &#x20;

* Create a **Public** OpenID Connect Client.&#x20;
  * OpenID Connect Client can be reffered to as OIDC Client or OAuth Client.
  * The technical term for Public OIDC Client is: Authorization Code Flow + PKCE.
  * If you are provided with the option, **disable client credentials,** you do not need to provide any client secret to oidc-sp&#x61;**.**
  * Some provider will simply ask you to select an Application type and let you chose between Single Page Application (SPA), Web Application (Or Web Server App) and Mobile. **Select SPA**.
* Valid Redirect URIs: **https://my-app.com/** and **http://localhost:5173/**
  * The slash at the end is important
  * If your app is hosted on a sub path like /dashboard you would set **https://my-app.com/dashboard/** and **http://localhost:5173/dashboard/**
  * 5173 is the default port used by Vite dev server, adapt according to your setup.
* Valid post logout redirect: Same as the Valid Redirec URIs
* Web Origins: **https://my-app.com**, **http://localhost:5173**
