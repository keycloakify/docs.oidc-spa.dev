---
icon: code-simple
---

# Keycloak Utils

oidc-spa is provider agnostic. You won't find in it's core any feature that will only work with Keycloak. &#x20;

However Keycloak is absolutly first class citizen when it come to keycloak integration, oidc-spa exposes utils to levrage keycloak specific features.

```typescript
import { createKeycloakUtils, isKeycloak } from "oidc-spa/keycloak";

const oidc = await getOidc(); // or useOidc() or inject(Oidc);

// Optional: Check at runtime is the app is a keycloak instance.
if( !isKeycloak({ issuerUri: oidc.issuerUri }) ){
    console.log("The authorization server is not a Keycloak Instance");
    return;
}

// keycloakUtils is a wrapper of pure, side effect free functions.
// createKeycloakUtils is memoized and non expensive to create.
const keycloakUtils = createKeycloakUtils({ issuerUri: oidc.issuerUri });

// Redirecting directly to the register pages instead of the login page.
oidc.login({
    doesCurrentHrefRequiresAuth: false,
    transformUrlBeforeRedirect: keycloakUtils.transformUrlBeforeRedirectForRegister
});

// url string to the Keycloak Account Console.
// Where the users can update their account information.
// See: https://docs.oidc-spa.dev/v/v9/features/user-account-management#redirecting-to-your-idps-account-managment-page
const accountUrl = keycloakUtils.getAccountUrl({
    clientId: oidc.clientId,
    validRedirectUri: oidc.validRedirectUri,
    locale: "en" // Optional
});

// Fetch Keycloak's internal representation of the user.
// Richer than the decodedIdToken
const userProfile = await keycloakUtils.fetchUserProfile({ 
    accessToken: await oidc.getAccessToken() // or (await oidc.getTokens()).accesToken
});
userProfile.id;
userProfile.username;
userProfile.userVerified;
userProfile.totp;
userProfile.attributes;
// ...

// Calling the well-known userinfo endpoint.
const userInfo = await keycloakUtils.fetchUserInfo({
    accessToken: await oidc.getAccessToken() // or (await oidc.getTokens()).accesToken
});
userInfo.sub;
// This is an object similar than what you get when you do:
const { decodeJwt } = await import("oidc-spa/decode-jwt");
const decodedAccessToken = decodeJwt(accessToken);

// url string to Keyloak's Admin Console
// You should first check if the user has "realm-admin" before displaying this link.
keycloakUtils.adminConsoleUrl; // Link to the admin console of the realm.
keycloakUtils.adminConsoleUrl_master; // Link to the master admin console.

const { issuerUriParsed } = keycloakUtils;
// eg: If issuerUri is "https://auth.my-company.com/realms/myrealm"
issuerUriParsed.origin; // eg: "https://auth.my-company.com"
issuerUriParsed.realm; // eg: "myrealm"
issuerUriParsed.kcHttpRelativePath; // eg: undefined or "/auth"
```

