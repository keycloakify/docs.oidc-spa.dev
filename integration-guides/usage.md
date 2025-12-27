---
icon: person-snowboarding
---

# Framework Agnostic Adapter

These are the instructions for setting up the framework-agnostic adapter for oidc-spa in a Single Page Application (SPA). These apps run entirely in the browser.

If your project uses Server-Side Rendering (SSR), this setup may not work. Don’t hesitate to [reach out on Discord](https://discord.gg/mJdYJSdcm4). We’re happy to help with your specific stack.

## Installation

{% stepper %}
{% step %}
### Installing the dependencies

{% tabs %}
{% tab title="npm" %}
```bash
npm install oidc-spa zod
```
{% endtab %}

{% tab title="yarn" %}
```bash
yarn add oidc-spa zod
```
{% endtab %}

{% tab title="pnpm" %}
```bash
pnpm add oidc-spa zod
```
{% endtab %}

{% tab title="bun" %}
```bash
bun add oidc-spa zod
```
{% endtab %}
{% endtabs %}

> **Note:**\
> [Zod](https://zod.dev/) is optional but highly recommended.\
> Writing validators manually is error-prone, and skipping validation means losing early guarantees about what your auth server provides.
{% endstep %}

{% step %}
### Global Setup

Pick one of these three options:

{% tabs %}
{% tab title="Vite Plugin" %}
If you're in a Vite project, the recomended approach is to use oidc-spa's Vite plugin.

<pre class="language-typescript" data-title="vite.config.ts"><code class="lang-typescript">import { defineConfig } from "vite";
<strong>import { oidcSpa } from "oidc-spa/vite-plugin";
</strong>
export default defineConfig({
    plugins: [
        // ...
<strong>        oidcSpa()
</strong>    ]
});
</code></pre>
{% endtab %}

{% tab title="Manual - Recommended" %}
Pick this approach if:

* You're not in a Vite project and
* Your app has a single client entrypoint.

***

Let's assume your app entrypoint is `src/main.ts`.

First, rename it to `src/main.lazy.ts`.

```bash
mv src/main.ts src/main.lazy.ts
```

Then create a new `src/main.ts` file:

{% code title="src/main.ts" %}
```typescript
import { oidcEarlyInit } from "oidc-spa/entrypoint";

const { shouldLoadApp } = oidcEarlyInit({
    BASE_URL: "/" // The path where your app is hosted. You can also pass it later to createOidc().
});

if (shouldLoadApp) {
    // Note: Deferring the main app import adds a few milliseconds to cold start,
    // but dramatically speeds up auth. Overall, it's a net win.
    import("./main.lazy");
}
```
{% endcode %}
{% endtab %}

{% tab title="Manual - Easy" %}
If you’re not using Vite and you can’t edit your app’s entry file, run `oidcEarlyInit()` in the same module where you call `createOidc()`.

<pre class="language-typescript" data-title="src/oidc.ts"><code class="lang-typescript">import { 
<strong>   oidcEarlyInit, 
</strong>   createOidc 
} from "oidc-spa/core";

// Should run as early as possible.  
<strong>oidcEarlyInit({ 
</strong><strong>   BASE_URL: "/" // The path where your app is hosted
</strong><strong>});
</strong>
const prOidc = createOidc({ /* ... See below ... */ });

export async function getOidc(){
   const oidc = await prOidc;
   return oidc;
}
</code></pre>
{% endtab %}
{% endtabs %}
{% endstep %}

{% step %}
### Initialize the adapter

This is just a suggestion. Feel free to adapt how you set things up.

{% code title="src/oidc.ts" %}
```typescript
import { createOidc } from "oidc-spa/core";
import { z } from "zod";

const prOidc = createOidc({
    // See: https://docs.oidc-spa.dev/v/v8/providers-configuration/provider-configuration
    issuerUri: "https://auth.your-domain.net/realms/myrealm",
    clientId: "myclient",

    // Optional. Expected shape of the ID token payload.
    // This is declarative. You describe what you will use and what
    // info you expect to be present in the ID token.
    // If you don't know what's in your ID token, open the console.
    // If you have debugLogs set to true, you'll see it.
    decodedIdTokenSchema: z.object({
       preferred_username: z.string(),
       name: z.string(),
       email: z.string().optional(),
       picture: z.string().optional(),
       realm_access: z.object({ roles: z.array(z.string()) }).optional()
    }),

    //scopes: ["profile", "email", "api://my-app/access_as_user"],

    // OPTIONAL, Parameters added when redirecting to the authorization endpoint.
    extraQueryParams: {
        //audience: "https://my-app.my-company.com/api",
        get ui_locales() { return "en"; } // Keycloak login/register pages language
    },

    debugLogs: true,
    
    // See: https://docs.oidc-spa.dev/v/v8/features/auto-login
    // autoLogin: true

    // See: https://docs.oidc-spa.dev/v/v8/features/dpop
    // dpop: "auto"
});

export async function getOidc(){
    const oidc = await prOidc;
    return oidc;
}
```
{% endcode %}
{% endstep %}
{% endstepper %}

## Usage

Here is a quick usage overview.

```typescript
import { getOidc } from "~/oidc"; // The file you created in the previous step

(async () => {
    const oidc = await getOidc();

    // oidc-spa exports Keycloak-specific utilities:
    const { createKeycloakUtils, isKeycloak } = await import("oidc-spa/keycloak");

    const keycloakUtils = isKeycloak({ issuerUri: oidc.params.issuerUri })
        ? createKeycloakUtils({ issuerUri: oidc.params.issuerUri })
        : undefined;

    // In oidc-spa the user is either logged in or they aren't.
    // The state will never mutate without a full app reload.
    if (oidc.isUserLoggedIn) {
        // The user is logged in.

        const {
            // The accessToken is what you'll use as a Bearer token to
            // authenticate to your APIs
            accessToken
        } = await oidc.getTokens();

        // oidc-spa also provides utilities to build API clients like this.
        fetch("https://api.your-domain.net/orders", {
            headers: {
                Authorization: `Bearer ${accessToken}`
            }
        })
            .then(response => response.json())
            .then(orders => console.log(orders));

        // Call when the user clicks logout.
        // You can also redirect to a custom URL with:
        // { redirectTo: "specific URL", url: "/bye" }
        oidc.logout({ redirectTo: "home" });

        const decodedIdToken = oidc.getDecodedIdToken();

        console.log(`Hello ${decodedIdToken.preferred_username}`);

        if (keycloakUtils) {
            // Get a link to the account page:
            const userAccountUrl = keycloakUtils.getAccountUrl({
                clientId: oidc.params.clientId,
                validRedirectUri: oidc.params.validRedirectUri
            });
        }
    } else {
        // The user is not logged in.

        // We can call login() to redirect the user to the login/register page.
        // This returns a promise that never resolves.
        oidc.login({
            /**
             * If you are calling login() in the callback of a click event
             * set this to false.
             * If you are calling this because the user has navigated to
             * a route that requires them to be logged in, set this to true.
             */
            doesCurrentHrefRequiresAuth: false,
            /**
             * Optionally, you can add extra parameters
             * to be added to the authorization endpoint.
             */
            //extraQueryParams: { kc_idp_hint: "google", ui_locales: "fr" }
            /**
             * You can also set where to redirect the user after
             * successful login but by default it's the current URL
             * which is usually what you want.
             */
            // redirectUrl: "/dashboard"
        });

        // Register button callback (Keycloak only)
        if (keycloakUtils) {
            oidc.login({
                doesCurrentHrefRequiresAuth: false,
                transformUrlBeforeRedirect: keycloakUtils.transformUrlBeforeRedirectForRegister
            });
        }
    }
})();
```

## Mock adapter

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

const prOidc = !import.meta.env.VITE_OIDC_ISSUER
<strong>    ? createMockOidc({
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
</strong>    : createOidc({
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

{% content-ref url="tanstack-router-+-node-rest-api/" %}
[tanstack-router-+-node-rest-api](tanstack-router-+-node-rest-api/)
{% endcontent-ref %}
