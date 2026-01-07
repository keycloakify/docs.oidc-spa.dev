---
icon: code-simple
---

# Keycloak Utils

oidc-spa is provider agnostic.\
You won’t find any Keycloak-only logic in the core package.

If you _are_ using Keycloak, `oidc-spa/keycloak` exposes small utilities to leverage Keycloak-specific URLs and endpoints.

{% hint style="info" %}
These utilities are **pure** (no side effects) and only need your `issuerUri`.\
`createKeycloakUtils()` is memoized, so it’s safe to call often.
{% endhint %}

### Import

```typescript
import { createKeycloakUtils, isKeycloak } from "oidc-spa/keycloak";
```

### Optional runtime check: is this issuer Keycloak?

Useful when your app can run against multiple providers.

```typescript
const oidc = await getOidc(); // or useOidc() or inject(Oidc)

if (!isKeycloak({ issuerUri: oidc.issuerUri })) {
    console.log("The authorization server is not a Keycloak instance");
    return;
}
```

### Create the utils object

```typescript
const keycloakUtils = createKeycloakUtils({ issuerUri: oidc.issuerUri });
```

### Common use cases

#### Redirect to the registration page (instead of login)

```typescript
oidc.login({
    doesCurrentHrefRequiresAuth: false,
    transformUrlBeforeRedirect: keycloakUtils.transformUrlBeforeRedirectForRegister
});
```

#### Link to the Keycloak Account Console

Users can update their profile, password, MFA, sessions, etc.

```typescript
const accountUrl = keycloakUtils.getAccountUrl({
    clientId: oidc.clientId,
    validRedirectUri: oidc.validRedirectUri,
    locale: "en" // Optional
});
```

See: [#redirecting-to-your-idps-account-managment-page](user-account-management.md#redirecting-to-your-idps-account-managment-page "mention")

#### Fetch the Keycloak user profile (Keycloak-internal endpoint)

This is richer than the decoded ID token.\
Equivalent of `keycloak-js` `.loadUserProfile()`.

```typescript
const accessToken = await oidc.getAccessToken(); // or (await oidc.getTokens()).accessToken

const userProfile = await keycloakUtils.fetchUserProfile({ accessToken });

userProfile.id;
userProfile.username;
userProfile.attributes;
```

#### Fetch user info (OIDC `userinfo` endpoint)

Equivalent of `keycloak-js` `.loadUserInfo()`.

```typescript
const accessToken = await oidc.getAccessToken(); // or (await oidc.getTokens()).accessToken

const userInfo = await keycloakUtils.fetchUserInfo({ accessToken });
userInfo.sub;
```

The userInfo object is similarly shaped as what you get if you decode the payload of the access token (which you shouldn't do on the client, see: [jwt-of-the-access-token.md](../resources/jwt-of-the-access-token.md "mention"))

```typescript
import { decodeJwt } from "oidc-spa/decode-jwt";
const decodedAccessToken = decodeJwt(accessToken);
```

#### Admin Console URLs

Only show these links to privileged users (for example `realm-admin`).

```typescript
keycloakUtils.adminConsoleUrl; // Admin console for the current realm
keycloakUtils.adminConsoleUrl_master; // Admin console for the "master" realm
```

### Parse the issuer URI

```typescript
const { issuerUriParsed } = keycloakUtils;

// Example issuerUri:
// "https://auth.my-company.com/realms/myrealm"
issuerUriParsed.origin; // "https://auth.my-company.com"
issuerUriParsed.realm; // "myrealm"
issuerUriParsed.kcHttpRelativePath; // undefined or "/auth" if the issuer uri was "https://auth.my-company.com/auth/realms/myrealm"
```
