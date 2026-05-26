---
icon: person-snowboarding
metaLinks:
  alternates:
    - https://app.gitbook.com/s/oygeayjvIPxroUcp3jt4/integration-guides/usage
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

{% include "../.gitbook/includes/setup-option.md" %}
{% endstep %}

{% step %}
### Initialize the adapter

This is just a suggestion. Feel free to adapt how you set things up.

{% code title="src/oidc.ts" %}
```typescript
import { createOidc } from "oidc-spa/core";

const prOidc = createOidc({
    // See: https://docs.oidc-spa.dev/v/v9/providers-configuration/provider-configuration
    issuerUri: "https://auth.your-domain.net/realms/myrealm",
    clientId: "myclient",

    //scopes: ["profile", "email", "api://my-app/access_as_user"],

    // OPTIONAL, Parameters added when redirecting to the authorization endpoint.
    extraQueryParams: {
        //audience: "https://my-app.my-company.com/api",
        get ui_locales() { return "en"; } // Keycloak login/register pages language
    },

    debugLogs: true,

    // See: https://docs.oidc-spa.dev/v/v9/features/auto-login
    // autoLogin: true

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

    const keycloakUtils = isKeycloak({ issuerUri: oidc.issuerUri })
        ? createKeycloakUtils({ issuerUri: oidc.issuerUri })
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

        // NOTE: We recomend implementing the user abstraction
        // over reading directly the ID token.
        // See: https://docs.oidc-spa.dev/v/v9/features/user
        const decodedIdToken = oidc.getDecodedIdToken();

        console.log(`Hello ${decodedIdToken.preferred_username}`);

        if (keycloakUtils) {
            // Get a link to the account page:
            const userAccountUrl = keycloakUtils.getAccountUrl({
                clientId: oidc.clientId,
                validRedirectUri: oidc.validRedirectUri,
                locale: "en" // Optional
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
<strong>import { createMockOidc } from "oidc-spa/core-mock";
</strong>// Optional, see: https://docs.oidc-spa.dev/v/v9/features/user
import { createUser, user_mock } from "./oidc.user";

const autoLogin = false;

const prOidc = !import.meta.env.VITE_OIDC_ISSUER
<strong>    ? createMockOidc({
</strong><strong>          // NOTE: If autoLogin is set to true this option must be removed
</strong><strong>          isUserInitiallyLoggedIn: false,
</strong><strong>          // Optional: 
</strong><strong>          mockedParams: {
</strong><strong>              issuerUri: "https://auth.my-company.com/realms/myrealm",
</strong><strong>              clientId: "myclient"
</strong><strong>          },
</strong><strong>          mockedUser: user_mock,
</strong><strong>          autoLogin
</strong><strong>      })
</strong>    : createOidc({
          issuerUri: import.meta.env.VITE_OIDC_ISSUER,
          clientId: import.meta.env.VITE_OIDC_CLIENT_ID,
          createUser,
          autoLogin
      });
</code></pre>

{% include "../.gitbook/includes/creating-an-api-server.md" %}
