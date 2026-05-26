---
icon: user
---

# The User Object

{% hint style="warning" %}
Work In Progress. This feature hasn't beeing released yet.
{% endhint %}

> Introduced in oidc-spa v10.3&#x20;

At your application level you typically have an object that represent the user that is currently using your application. &#x20;

For example:

```typescript
export type User = {
    id: string;
    username: string;
    displayName: string;
    email: string | undefined;
    avatarImgUrl: string;
    hasRole: (role: string) => boolean;
};
```

The iformations for costructing the desired user object can comes from different sources:

* The Decoded ID Token
* The Decoded Access Token (even if, in theory, the access token is supposed to be opaque for the SPA, the roles of the users are often only available in the JWT payload of the access token)
* By querying a custom `/api/user` endpoind of your API with an access token as bearer.&#x20;
* By calling the standard userinfo OIDC endpoint.&#x20;
* By calling provider specific endpoints like keycloak's user profile.

oidc-spa adapters let you decide what the user should look like (by providing your own type definition for the User object) and how it should be created, by letting you implement a createUser function that is called with all the material that you might need to create the user object. &#x20;



The first thing you need to do is to declare the desired shape of the app level user object and implement a function to create that user object (this should be framwork agnostic):

{% code title="src/oidc.user.ts" %}
```typescript
import type { CreateUser } from "oidc-spa/core";
import { z } from "zod";
import avatarFallbackSvgUrl from "./assets/avatarFallback.svg";

// App-level user shape exposed by `useOidc()`.
// You decide what an user should looks like!
export type User = {
    id: string;
    username: string;
    displayName: string;
    email: string | undefined;
    avatarImgUrl: string;
    isRealmAdmin: boolean;
    userInfo: {
        sub: string;
        [claim: string]: unknown;
    };
    keycloakUserProfile?: import("oidc-spa/keycloak").KeycloakProfile;
};

// The function that oidc-spa will call to create the user object,
// gathering information from different sources depending of what you need.
export const createUser: CreateUser<User> = async ({
    decodedIdToken: decodedIdToken_generic,
    accessToken,
    fetchUserInfo,
    issuerUri
}) => {
    /* ================= Possible source: ID token claims. ====================== */

    const DecodedIdToken = z.object({
        sub: z.string(),
        name: z.string(),
        picture: z.string().optional(),
        email: z.string().email().optional(),
        preferred_username: z.string().optional()
    });

    const decodedIdToken = DecodedIdToken.parse(decodedIdToken_generic);

    /* ================== Possible source: access token claims. ================== */
    // This is pragmatic, but not textbook OIDC: clients should usually
    // treat access tokens as opaque, and some providers do not issue JWTs.

    const DecodedAccessToken = z.object({
        realm_access: z.object({ roles: z.array(z.string()) }).optional()
    });

    const { decodeJwt } = await import("oidc-spa/decode-jwt");
    const { isKeycloak } = await import("oidc-spa/keycloak");

    const decodedAccessToken = !isKeycloak({ issuerUri })
        ? undefined
        : DecodedAccessToken.parse(decodeJwt(accessToken));

    /* ================= Possible source: your own API. ========================= */

    // const { fetchWithAuth } = await import("./oidc");
    // const userFromApi = await fetchWithAuth("/api/user").then(r => r.json());

    /* ================= Possible source: the standard OIDC UserInfo endpoint. == */

    const userInfo = await fetchUserInfo();

    /* ================= Possible source: provider-specific endpoints. ========== */
    const { createKeycloakUtils } = await import("oidc-spa/keycloak");

    const keycloakUtils = isKeycloak({ issuerUri }) ? 
        createKeycloakUtils({ issuerUri }) : undefined;

    const keycloakUserProfile = await keycloakUtils?.fetchUserProfile({ accessToken });

    /* ================== Merging =============================================== */
    // Merge whichever sources you decided to use into the single
    // `User` shape consumed by the rest of the app.

    const user: User = {
        id: decodedIdToken.sub,
        username: decodedIdToken.preferred_username ?? decodedIdToken.sub,
        displayName: decodedIdToken.name,
        avatarImgUrl: decodedIdToken.picture || avatarFallbackSvgUrl,
        email: decodedIdToken.email,
        isRealmAdmin: decodedAccessToken?.realm_access?.roles.includes("realm-admin") ?? false,
        userInfo,
        keycloakUserProfile
    };

    return user;
};

// App-level user returned when the mock implementation is enabled.
export const user_mock: User = {
    id: "mock-user",
    username: "john.doe",
    displayName: "John Doe",
    email: undefined,
    avatarImgUrl: avatarFallbackSvgUrl,
    isRealmAdmin: true,
    userInfo: { sub: "1234" },
    keycloakUserProfile: undefined
};
```
{% endcode %}



{% tabs %}
{% tab title="Framwork Agnostic" %}
<pre class="language-typescript" data-title="src/oidc.ts"><code class="lang-typescript">import { createOidc } from "oidc-spa/core";
import { createUser, user_mock } from "./oidc.user";
import { createMockOidc } from "oidc-spa/core-mock";

const autoLogin = false;

export const prOidc = !import.meta.env.VITE_OIDC_ISSUER
    ? createMockOidc({
          // NOTE: If autoLogin is set to true this option must be removed
          isUserInitiallyLoggedIn: false,
          // Optional:
          mockedParams: {
              issuerUri: "https://auth.my-company.com/realms/myrealm",
              clientId: "myclient"
          },
<strong>          mockedUser: user_mock,
</strong>          autoLogin
      })
    : createOidc({
          issuerUri: import.meta.env.VITE_OIDC_ISSUER,
          clientId: import.meta.env.VITE_OIDC_CLIENT_ID,
<strong>          createUser,
</strong>          autoLogin
      });

<strong>export async function greatUser() {
</strong><strong>    const oidc = await prOidc;
</strong><strong>
</strong><strong>    if (!oidc.isUserLoggedIn) {
</strong><strong>        return;
</strong><strong>    }
</strong><strong>
</strong><strong>    const { user } = await oidc.getUser();
</strong><strong>
</strong><strong>    alert(`Hello ${user.displayName}`);
</strong><strong>}
</strong></code></pre>
{% endtab %}

{% tab title="React" %}
<pre class="language-typescript" data-title="src/oidc.ts"><code class="lang-typescript">import { oidcSpa } from "oidc-spa/react-spa";
                  // or "oidc-spa/react-tanstack-start"
<strong>import { type User, createUser, user_mock } from "./oidc.user";
</strong>
export const {
    bootstrapOidc,
    useOidc,
    getOidc,
    enforceLogin,
    OidcInitializationGate
} = oidcSpa
<strong>    .withUser&#x3C;User>({ createUser, user_mock })
</strong>    .createUtils();

bootstrapOidc(
    import.meta.env.VITE_OIDC_USE_MOCK === "true"
        ? {
              implementation: "mock",
              isUserInitiallyLoggedIn: true
<strong>              // You can also override `user_mock` here.
</strong>          }
        : {
              implementation: "real",
              issuerUri: import.meta.env.VITE_OIDC_ISSUER_URI,
              clientId: import.meta.env.VITE_OIDC_CLIENT_ID,
          }
);
</code></pre>

Usage:

```tsx
import { useOidc, getOidc } from "~/oidc";

// Accessing the user object in a react component.
function Hero() {
    const { user } = useOidc({ assert: "user logged in" });

    return <h1>Hello {user.displayName}!</h1>;
}

// ... and outside react:
async function greetUser(){

    const oidc = await getOidc({ assert: "user logged in" });

    const { user } = await oidc.getUser();

    alert(`Hello ${user.displayName}`);

}
```
{% endtab %}
{% endtabs %}

