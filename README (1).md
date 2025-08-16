---
icon: up
---

# v5 -> v6

Here’s what’s new in version 6 of `oidc-spa`:

* **Full compatibility with any OIDC provider**\
  In v5, you had to define valid redirect URIs using wildcards, such as `https://my-app.com/dashboard/*`. However, according to the OIDC specification, wildcards in redirect URIs are not allowed. Some OIDC servers, like Ory Hydra, enforce this rule, as discussed [here](https://github.com/ory/hydra/discussions/2512).\
  In v6, you only need to define a single redirect URI, the homepage of your app (e.g., `https://my-app.com/dashboard/`).
* **Enhanced security: No more token storage in session storage**\
  Tokens are no longer stored in session storage, aligning with modern security best practices.
* **Eliminated the need for a `silent-sso.htm` file**\
  Making oidc-spa self contained and easyer to setup.
* **Improved error messages**\
  If something is misconfigured, error messages now provide much clearer explanations of the root cause.
* **API refinements**\
  Several API improvements aiming at making the library usage more intuitive.

## Migration Guide

### 1. Remove `public/silent-sso.htm`

The silent SSO file is no longer needed, so it should be deleted from your project.

### 2. Update configuration changes

The following changes have been made to the API:

{% code title="src/oidc.ts" %}
```diff
export const { OidcProvider, useOidc, getOidc } = createReactOidc({
    issuerUri: "https://auth.your-domain.net/realms/myrealm",
    clientId: "myclient",
    // `publicUrl` has been renamed to `homeUrl`
-   publicUrl: import.meta.env.BASE_URL,
+   homeUrl: import.meta.env.BASE_URL,
    // `isAuthGloballyRequired` has been renamed to `autoLogin`
-   isAuthGloballyRequired: true,
+   autoLogin: true,
    // `doEnableDebugLogs` has been renamed to `debugLogs`
-   doEnableDebugLogs: true,
+   debugLogs: true,
});

createMockReactOidc({
   // ...
-  publicUrl: import.meta.env.BASE_URL,
+  homeUrl: import.meta.env.BASE_URL,
});
```
{% endcode %}

### 3. Update authentication assertion

The `assertUserLoggedIn` option has been replaced:

```diff
-const { oidcTokens } = useOidc({ assertUserLoggedIn: true });
+const { oidcTokens } = useOidc({ assert: "user logged in" });
```

### 4. (OPTIONAL, Recommended) Update your usage of the useOidc hook

{% code title="Before" %}
```tsx

const { oidcTokens } = useOidc({ assertUserLoggedIn: true });

<span>{oidcTokens.decodedIdToken.preffered_username}</span>

```
{% endcode %}

{% code title="After" %}
```tsx
const { decodedIdToken } = useOidc({ assert: "user logged in" });

<span>{decodedIdToken.preffered_username}</span>
```
{% endcode %}

***

{% code title="Before" %}
```tsx
const { oidcTokens } = useOidc({ assertUserLoggedIn: true });

useEffect(()=> {
    fetch(
        "https://...", 
        { headers: { "Auhtorization": `Bearer ${oidcToken.accessToken}` }}
    );
}, []);
```
{% endcode %}

Recomendation is to declare an [fetchWithAuthorization like in the example](https://app.gitbook.com/s/u20Nc4nUTlX9s50rXkBi/usage#react-api), but if you want the direct equivalent:

{% code title="After" %}
```typescript
const { tokens } = useOidc({ assert: "user logged in" });

useEffect(()=> {
    // Token will always be undefined the first render.
    if( tokens === undefined ) return;
    fetch(
        "https://...", 
        { headers: { "Auhtorization": `Bearer ${tokens.accessToken}` }}
    );
}, []);
```
{% endcode %}

### 5. (OPTIONAL) Assume getTokens() is an async function

In the next major getTokens() will be async. So if you want to be able to migrate without issue, start assuming it is today:

```diff
export const fetchWithAuth: typeof fetch = async (
    input,
    init
) => {
    const oidc = await getOidc();

    if (oidc.isUserLoggedIn) {
-       const { accessToken } = oidc.getTokens();
+       const { accessToken } = await oidc.getTokens();

        (init ??= {}).headers = {
            ...init.headers,
            Authorization: `Bearer ${accessToken}`
        };
    }

    return fetch(input, init);
};
```

### 6. Error Management Updates

* `OidcInitializationError` now only includes the `isAuthServerLikelyDown` property, which is `true` if the authentication server is likely down.\
  If it’s `false`, the OIDC server is reachable, but there is a client/server missconfiguration.
* The `initializationError.type` property has been **removed**.

{% content-ref url="https://app.gitbook.com/s/u20Nc4nUTlX9s50rXkBi/error-management" %}
[Error Management](https://app.gitbook.com/s/u20Nc4nUTlX9s50rXkBi/error-management)
{% endcontent-ref %}

### 7. Keycloak Configuration Improvements

The Keycloak setup guide has been updated for better clarity:

{% content-ref url="https://app.gitbook.com/s/u20Nc4nUTlX9s50rXkBi/providers-configuration/keycloak" %}
[Keycloak](https://app.gitbook.com/s/u20Nc4nUTlX9s50rXkBi/providers-configuration/keycloak)
{% endcontent-ref %}

### 8. Session Initialization Changes

* The `authMethod` option has been **removed**.
* The `isNewBrowserSession` property should now be used instead.

{% content-ref url="https://app.gitbook.com/s/u20Nc4nUTlX9s50rXkBi/user-session-initialization" %}
[User Session Initialization](https://app.gitbook.com/s/u20Nc4nUTlX9s50rXkBi/user-session-initialization)
{% endcontent-ref %}

### 9. Impersonation

oidc-spa v5 had built in [impersonation cappability](https://docs.oidc-spa.dev/docs/v5/documentation/user-impersonation).  \
However, this mechanism was somewhat hard to implement in practice, and could expose your app to vulnerabilities if not implemented correctly.  \
If you want to implement impersonation, here is an alternative approach for Keycloak: &#x20;

{% embed url="https://github.com/keycloakify/keycloakify-starter/tree/direct_impersonation" %}

If you where using this feature, just reach out to me, I'm open to re-introducing it. &#x20;
