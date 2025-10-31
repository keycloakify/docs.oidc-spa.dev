---
description: Let's get your App authenticated!
icon: person-snowboarding
---

# Framwork Agnostic

{% hint style="info" %}
oidc-spa is framework agnostic indeed BUT it's a client centric library. &#x20;

Meaning that it will integrate seamlessly with any Single Page Application but not with full stack framwork like Next that involve SSR.

The only full stack framework supported is TanStack Start.
{% endhint %}

{% hint style="info" %}
Note that if you are using React but not TanStack not React Router you'll still probably be better served by oidc-spa/react-spa than oidc-spa/core. If it's your case follow the React-Router integration path in "Declarative Mode", you should be able to adapt to your environnement.
{% endhint %}

## Installation

{% tabs %}
{% tab title="npm" %}
```bash
npm install oidc-spa
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

To protect tokens against supply-chain attacks and XSS, oidc-spa must run some initialization code _before any other JavaScript in your app_.

This design provides much stronger security guarantees than any other adapter, and it also delivers unmatched login performance. More details [here](../why-oidcearlyinit.md).

{% tabs %}
{% tab title="Vite SPAs" %}
In Vite apps, this is done through a Vite Plugin (If you'd rather avoid using the Vite plugin checkout the Other SPAs tab).

<pre class="language-typescript" data-title="vite.config.ts"><code class="lang-typescript">import { reactRouter } from "@react-router/dev/vite";
import { defineConfig } from "vite";
import tsconfigPaths from "vite-tsconfig-paths";
<strong>import { oidcSpa } from "oidc-spa/vite-plugin";
</strong>
export default defineConfig({
    plugins: [
        reactRouter(),
<strong>        oidcSpa({
</strong><strong>            freezeFetch: true,
</strong><strong>            freezeXMLHttpRequest: true,
</strong><strong>            freezeWebSocket: true
</strong><strong>        }),
</strong>        tsconfigPaths()
    ]
});
</code></pre>
{% endtab %}

{% tab title="Other SPAs" %}
First rename your entry point file from `main.tsx` (or `main.ts` or whatever it is) to `main.lazy.tsx`

```bash
mv src/main.ts src/main.lazy.ts
```

Then create a new `index.tsx` file:

{% code title="src/index.tsx" %}
```typescript
import { oidcEarlyInit } from "oidc-spa/entrypoint";

const { shouldLoadApp } = oidcEarlyInit({
    freezeFetch: true,
    freezeXMLHttpRequest: true,
    freezeWebSocket: true,
    BASE_URL: "/" // The path where your app is hosted, can also be provided later to createOidc()
});

if (shouldLoadApp) {
    // Note: Deferring the main app import adds a few milliseconds to cold start,
    // but dramatically speeds up auth. Overall, it's a net win.
    import("./index.lazy");
}
```
{% endcode %}

If you don't have a precise entrypoint that you can simply override, just call oidcEarlyInit as soon as possible and try canceling as much work as possible when `shouldLoadApp` is false.
{% endtab %}
{% endtabs %}

## Basic Usage

```typescript
import { createOidc } from "oidc-spa/core";
import { z } from "zod";

const oidc = await createOidc({
    issuerUri: "https://auth.your-domain.net/realms/myrealm",
    clientId: "myclient",
    //scopes: ["profile", "email", "api://my-app/access_as_user"],
    extraQueryParams: () => ({
       ui_locales: "en" // Keycloak login/register pages language
       //audience: "https://my-app.my-company.com/api"
    }),
    // This is declarative, you declare what you will use and what
    // infos you expect to be present in the id token.
    // if you don't know what's in your id token open the console
    // if you have debugLogs set to true you'll see.
    decodedIdTokenSchema: z.object({
       preferred_username: z.string(),
       name: z.string()
       email: z.string().email().optional(),
       picture: z.string().optional(),
       email: z.string().email().optional(),
       realm_access: z.object({ roles: z.array(z.string()) }).optional()
    }),
    debugLogs: true
});

if (!oidc.isUserLoggedIn) {
    // The user is not logged in.

    // We can call login() to redirect the user to the login/register page.
    // This return a promise that never resolve. 
    oidc.login({
         /** 
          * If you are calling login() in the callback of a click event
          * set this to false.  
          * If you are calling this because the user has navigated to
          * a route that requires them to be logged in, set this to true.
          */
         doesCurrentHrefRequiresAuth: false
         /** 
          * Optionally, you can add some extra parameter 
          * to be added on the login url.  
          * (Can also be a parameter of createOidc `extraQueryParams: ()=> ({ ui_locales: "fr" })`)
          */
         //extraQueryParams: { kc_idp_hint: "google", ui_locales: "fr" }
         /**
          * You can allso set where to redirect the user after 
          * successful login but by default it's the current url
          * which is usually what you want.
          */
          // redirectUrl: "/dashboard"
          
          /**
           * Keycloak: You can also send the users directly to the register page
           * see: https://github.com/keycloakify/oidc-spa/blob/14a3777601c50fa69d1221495d77668e97443119/examples/tanstack-router-file-based/src/components/Header.tsx#L54-L66
           */ 
    });

} else {
    // The user is logged in.

    const {
        // The accessToken is what you'll use as a Bearer token to 
        // authenticate to your APIs
        accessToken
    } = await oidc.getTokens();
    
    // oidc-spa also provide the toold to create such an API.
    fetch("https://api.your-domain.net/orders", {
        headers: {
            Authorization: `Bearer ${accessToken}`
        }
    })
     .then(response => response.json())
     .then(orders => console.log(orders));

    // To call when the user click on logout.
    // You can also redirect to a custom url with 
    // { redirectTo: "specific url", url: "/bye" }
    oidc.logout({ redirectTo: "home" });
    
    const decodedIdToken = oidc.getDecodedIdToken();

    console.log(`Hello ${decodedIdToken.preferred_username}`);

}
```

## Mock Adapter

For certain use cases, you may want a mock adapter to simulate user authentication without involving an actual authentication server.

This approach is useful when building an app where user authentication is a feature but not a requirement. It also proves beneficial for running tests or in Storybook environments.

<pre class="language-typescript"><code class="lang-typescript">import { createOidc } from "oidc-spa/core";
<strong>import { createMockOidc } from "oidc-spa/mock";
</strong>import { z } from "zod";

const decodedIdTokenSchema = z.object({
    sub: z.string(),
    preferred_username: z.string()
});

const autoLogin = false;

const oidc = !import.meta.env.VITE_OIDC_ISSUER
<strong>    ? await createMockOidc({
</strong><strong>          // NOTE: If autoLogin is set to true this option must be removed
</strong><strong>          isUserInitiallyLoggedIn: false,
</strong><strong>          mockedTokens: {
</strong><strong>              decodedIdToken: {
</strong><strong>                  sub: "123",
</strong><strong>                  preferred_username: "john doe"
</strong><strong>              } satisfies z.infer&#x3C;typeof decodedIdTokenSchema>
</strong><strong>          },
</strong><strong>          autoLogin
</strong><strong>      })
</strong>    : await createOidc({
          issuerUri: import.meta.env.VITE_OIDC_ISSUER,
          clientId: import.meta.env.VITE_OIDC_CLIENT_ID,
          decodedIdTokenSchema,
          autoLogin
      });
</code></pre>

## Creating an API server

Now that authentication is handled, there’s one last piece of the puzzle: your resource server, the backend your app will communicate with.

This can be any type of service: a REST API, tRPC server, or WebSocket endpoint, as long as it can validate access tokens issued by your IdP.

If you’re building it in JavaScript or TypeScript (for example, using Express), oidc-spa provides ready-to-use utilities to decode and validate access tokens on the server side.

You’ll find the full documentation here:

{% content-ref url="tanstack-router-+-node-rest-api.md" %}
[tanstack-router-+-node-rest-api.md](tanstack-router-+-node-rest-api.md)
{% endcontent-ref %}
