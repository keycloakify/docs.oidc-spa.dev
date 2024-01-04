---
description: Let's get your web app authenticated in less than 20 minutes!
---

# 🚀 Getting started

{% hint style="info" %}
In this guide, we assume that you have an OIDC-enabled authentication server in place, such as Keycloak.&#x20;

If you have not yet set up such a server, please refer to [our guide for instructions on how to provision and configure a Keycloak server](usage-with-keycloak.md).
{% endhint %}

Let's see how to install oidc-spa in your [Vite](https://vitejs.dev/) or [Create React App](https://create-react-app.dev/) project: &#x20;

{% tabs %}
{% tab title="npm" %}
```bash
npm install --save oidc-spa
```
{% endtab %}

{% tab title="yarn" %}
```bash
yarn add oidc-spa
```
{% endtab %}

{% tab title="pnpm" %}
```bash
pnpm add oidc-spa
```
{% endtab %}

{% tab title="bun" %}
```bash
bun add oidc-spa
```
{% endtab %}
{% endtabs %}

Create the following file in your public directory: &#x20;

{% code title="public/silent-sso.html" %}
```html
<html>
    <body>
        <script>
            parent.postMessage(location.href, location.origin);
        </script>
    </body>
</html>
```
{% endcode %}

{% tabs %}
{% tab title="Usage without involving your UI framwork " %}
This is the approach you want to implement if your app implement strict separation of concern between your core logic and your UI components. &#x20;

```typescript
import { createOidc, decodeJwt } from "oidc-spa";

const oidc = await createOidc({
    issuerUri: "https://auth.your-domain.net/auth/realms/myrealm",
    clientId: "myclient",
    /**
     * Optional, you can modify the url before redirection to the identity 
     * server. Alternatively you can use: 
     * getExtraQueryParams: ()=> ({ ui_locales: "fr" })
     */
    transformUrlBeforeRedirect: url => `${url}&ui_locales=fr`
    /**
     * This parameter have to be provided if your App is not hosted at the 
     * origin of the domain.
     * For example if your site is accessed by navigating to 
     * https://www.example.com you don't have to provide this parameter.
     * On the other end if your site is accessed by navigating to 
     * https://www.example.com/my-app
     * Then you want to set publicUrl to `/my-app` 
     * (or `https://www.example.com/my-app`).
     
     * If you are using Vite: `publicUrl: import.meta.env.BASE_URL`
     * If you using Create React App: `publicUrl: process.env.PUBLIC_URL`
     *
     * Be mindful that `${window.location.origin}${publicUrl}/silent-sso.html` 
     * must return the `silent-sso.html` that you have created in the previous
     * step.
     */
    //publicUrl: import.meta.env.BASE_URL
});

if (!oidc.isUserLoggedIn) {
    // This return a promise that never resolve. 
    // The user will be redirected to the identity server.
    oidc.login({
         /** 
          * doesCurrentHrefRequiresAuth determines the behavior when a user 
          * gives up on loggin in and navigate back.
          * When it happens we don't want to send him back to a page that
          * can be accessed without authentication.
          *
          * If you are calling login() as a result of the user clicking
          * on a 'login' button (like here) you should set 
          * doesCurrentHrefRequiresAuth to false.
          *
          * When you are calling login because your user navigated to a path 
          * that requires authentication you should set 
          * doesCurrentHrefRequiresAuth to true
          */
         doesCurrentHrefRequiresAuth: false
         /** Optionally, you can add some extra parameter 
          *  to be added on the login url.
          */
         //extraQueryParams: { kc_idp_hint: "google" }
    });
} else {
    const {
        // The accessToken is what you'll use as a Bearer token to 
        // authenticate to your APIs
        accessToken,
        // You can parse the idToken as a JWT to get some information 
        // about the user.
        idToken
    } = oidc.getTokens();

    // NOTE: In most application you do not need to look into the JWT of the 
    // idToken on the frontend, you usually obtain the user info by querying 
    // a GET /user endpoint with a authorization header like 
    // `Bearer <accessToken>`.
    const user = decodeJwt(idToken) as {
        // Use https://jwt.io/ to tell what's in your idToken
        sub: string;
        preferred_username: string;
    };

    console.log(`Hello ${user.preferred_username}`);

    // To call when the user click on logout.
    // You can also redirect to a custom url with 
    // { redirectTo: "specific url", url: `${location.origin}/bye` }
    oidc.logout({ redirectTo: "home" });
}
```
{% endtab %}

{% tab title="Usage directly within React" %}
This piece of code should give you the necessary information to understand how oidc-spa can be used inside your react components.  \
To go further you can refer to the examples setup to see how to integrate oidc-spa with your routing library: &#x20;

* [react-router-dom example setup](https://github.com/garronej/oidc-spa/tree/main/examples/react-router)
* [@tanstack/react-router example setup](https://github.com/garronej/oidc-spa/tree/main/examples/tanstack-router)

```tsx
import { createOidcProvider, createUseOidc } from "oidc-spa/react";
import { z } from "zod";

const { OidcProvider } = createOidcProvider({
    issuerUri: "https://auth.your-domain.net/auth/realms/myrealm",
    clientId: "myclient",
    /**
     * Optional, you can modify the url before redirection to the identity 
     * server. Alternatively you can use: 
     * getExtraQueryParams: ()=> ({ ui_locales: "fr" })
     */
    transformUrlBeforeRedirect: url => `${url}&ui_locales=fr`
    /**
     * This parameter have to be provided if your App is not hosted at the 
     * origin of the domain.
     * For example if your site is accessed by navigating to 
     * https://www.example.com you don't have to provide this parameter.
     * On the other end if your site is accessed by navigating to 
     * https://www.example.com/my-app
     * Then you want to set publicUrl to `/my-app` 
     * (or `https://www.example.com/my-app`).
     
     * If you are using Vite: `publicUrl: import.meta.env.BASE_URL`
     * If you using Create React App: `publicUrl: process.env.PUBLIC_URL`
     *
     * Be mindful that `${window.location.origin}${publicUrl}/silent-sso.html` 
     * must return the `silent-sso.html` that you have created in the previous
     * step.
     */
    //publicUrl: import.meta.env.BASE_URL
});

const { useOidc } = createUseOidc({
    /**
     * This parameter is optional.
     * It allows you to validate the shape of the idToken so that you
     * can trust that oidcTokens.decodedIdToken is of the expected shape
     * when the user is logged in.
     * What is actually inside the idToken is defined by the OIDC server
     * you are using.
     * If you are not sure, you can copy the content of oidcTokens.idToken
     * and paste it on https://jwt.io/ to see what is inside.
     *
     * The usage of zod here is just an example, you can use any other schema
     * validation library or write your own validation function.
     *
     * If you want to specify the type of the decodedIdToken but do not care
     * about validating the shape of the decoded idToken at runtime you can
     * call `createUseOidc<DecodedIdToken>()` without passing any parameter.
     *
     * Note however that in most webapp you do not need to look into the JWT
     * of the idToken on the frontend side, you usually obtain the user info
     * by querying a GET /user endpoint with a authorization header
     * like `Bearer <accessToken>`.
     * If you don't use the decodedIdToken just do:
     * `export const { useOidc } = createUseOidc()`
     */
    decodedIdTokenSchema: z.object({
        sub: z.string(),
        preferred_username: z.string()
    })
});

ReactDOM.render(
    <OidcProvider
        // Optional, it's usually so fast that a fallback is really not required.
        fallback={<>Checking authentication ⌛️</>}
    >
        <App />
    </OidcProvider>,
    document.getElementById("root")
);

function App() {

    const { isUserLoggedIn, login, logout, oidcTokens } = useOidc();

    return (
        isUserLoggedIn ? (
            <>
                <span>Hello {oidcTokens.decodedIdToken.preferred_username}</span>
                <button onClick={() => logout({ redirectTo: "home" })}>
                  Logout
                </button>
            </>
        ) : (
            <button onClick={() => login({ 
              /** 
               * doesCurrentHrefRequiresAuth determines the behavior when a user 
               * gives up on loggin in and navigate back.
               * When it happens we don't want to send him back to a page that
               * can be accessed without authentication.
               *
               * If you are calling login() as a result of the user clicking
               * on a 'login' button (like here) you should set 
               * doesCurrentHrefRequiresAuth to false.
               *
               * When you are calling login because your user navigated to a path 
               * that requires authentication you should set 
               * doesCurrentHrefRequiresAuth to true
               */
              doesCurrentHrefRequiresAuth: false
              /** Optionally, you can add some extra parameter 
               *  to be added on the login url.
               */
              //extraQueryParams: { kc_idp_hint: "google" }
            })} >
              Login
            </button>
        )
    );
}
```
{% endtab %}
{% endtabs %}

### Refreshing token

The token refresh is handled automatically for you, however you can manually trigger a token refresh with `oidc.renewTokens()`.(or `const { renewTokens } = useOidc({ assertUserLoggedIn: true }`);&#x20;

### What happens if the OIDC server is down?

If the OIDC server is down or misconfigured an error get printed in the console, everything continues as normal with the user unauthenticated. If the user tries to login an alert saying that authentication is not available at the moment is displayed and nothing happens.\
This enable your the part of your app that do not requires authentication to remain up even when your identities server is facing issues.
