---
description: Let's get your App authenticated!
icon: person-snowboarding
---

# Framework Agnostic

{% hint style="info" %}
oidc-spa is framework-agnostic, but it’s also client-centric.

It integrates seamlessly with any Single Page Application, but not with full-stack frameworks that rely on server-side rendering (like Next.js).

The only full-stack framework currently supported is TanStack Start, which aligns with oidc-spa’s client-first, server capable architecture.
{% endhint %}

{% hint style="info" %}
Note that if you’re using **React** but not **TanStack** or **React Router,** you’ll likely still benefit more from `oidc-spa/react-spa` than from `oidc-spa/core`.

In that case, follow the [React Router integration guide in Declarative Mode](../setup-guides/react-router.md#declarative-mode), it should be easy to adapt to your setup.
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

This design provides much stronger security guarantees than any other adapter, and it also delivers unmatched login performance. More details [here](broken-reference).

{% tabs %}
{% tab title="Vite SPAs" %}
In Vite apps, this is done through a Vite Plugin (If you'd rather avoid using the Vite plugin checkout the Other SPAs tab).

<pre class="language-typescript" data-title="vite.config.ts"><code class="lang-typescript">import { defineConfig } from "vite";
<strong>import { oidcSpa } from "oidc-spa/vite-plugin";
</strong>
export default defineConfig({
    plugins: [
        // ...
<strong>        oidcSpa({
</strong><strong>            freezeFetch: true,
</strong><strong>            freezeXMLHttpRequest: true,
</strong><strong>            freezeWebSocket: true
</strong><strong>        })
</strong>    ]
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
    });
    
    // oidc-spa export keycloak specific tooling:
    const { isKeycloak, createKeycloakUtils } = await import("oidc-spa/keycloak");
    
    // If your IdP is a Keycloak
    if( isKeycloak({ issuerUri: oidc.issuerUri }) ){
        const keycloakUtils = createKeycloakUtils({ issuerUri: oidc.params.issuerUri });
        // Redirect directly to the register page instead of the login page
        oidc.login({
            doesCurrentHrefRequiresAuth: false,
            transformUrlBeforeRedirect: keycloakUtils.transformUrlBeforeRedirectForRegister
        });
        
    }
    

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
    
    // Get a link to the account page:
    const userAccountUrl = keycloakUtils.getAccountUrl({ 
        clientId: oidc.params.issuerUri,
        validRedirectUri: oidc.params.validRedirectUri
    });
    
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
