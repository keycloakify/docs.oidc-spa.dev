---
icon: up
---

# v5 -> v6

Here are what's new in the v6 of oidc-spa:

* Real support with any OIDC provider. The problem v5 had was that it forced you to be able to define Valid Redirect URIs with wildchard. Like for example https://my-app.com/dashboard/\*.  \
  According to the OIDC spec, having wildchard in the context of redirect URIs is not OK and some OIDC Server like Ory Hydra does not allow it as discussed [here](https://github.com/ory/hydra/discussions/2512). In v6 there is only the need to define a single redirect URI: the homepage of your app. Like for example https://my-app.com/dashboard/
* We stop storing tokens in the local session storage, aligning with the higher security standards.
* We got rid of the need to rely on a silent-sso.htm file.
* Mutch better error message: If something's wrong in your setup the quality of the message that exploains what's the cause have been greatly improved.
* Overall improvement of the API quality.

## Migration guide

First thing you want to do is remove the public/silent-sso.htm

{% code title="src/oidc.ts" %}
```diff
 export const { OidcProvider, useOidc, getOidc } = createReactOidc({
     issuerUri: "https://auth.your-domain.net/realms/myrealm",
     clientId: "myclient",
     // publicUrl renamed to homeUrl
-    publicUrl: import.meta.env.BASE_URL,
+    homeUrl: import.meta.env.BASE_URL,
     // isAuthGloballyRequired renamed to autoLogin
-    isAuthGloballyRequired: true,
+    autoLogin: true,
     // doEnableDebugLogs renamed to debugLogs
-    doEnableDebugLogs: true,
+    debugLogs: true,

 });
 
 createMockReactOidc({
    // ...
-    publicUrl: import.meta.env.BASE_URL,
+    homeUrl: import.meta.env.BASE_URL,
 });
```
{% endcode %}

assertUserLoggedIn is now specified differently: &#x20;

```diff
-const { oidcTokens } = useOidc({ assertUserLoggedIn: true });
+const { oidcTokens } = useOidc({ assert: "user logged in" });
```

### Error managment

The OidcInitializationError class has changed. Now it only has a property isAuthServerLikelyDown that is true when it's possible that the server is actually down.  \
If it's false, the OIDC Server seems to be up but there is something wrong in your client/server configuration. &#x20;

`initializationError.type` have been removed.\
\
Learn more: &#x20;

{% content-ref url="https://app.gitbook.com/s/u20Nc4nUTlX9s50rXkBi/error-management" %}
[Error Management](https://app.gitbook.com/s/u20Nc4nUTlX9s50rXkBi/error-management)
{% endcontent-ref %}

The Keycloak configuration guide has also been improved: &#x20;

{% content-ref url="https://app.gitbook.com/s/u20Nc4nUTlX9s50rXkBi/resources/keycloak-configuration" %}
[Keycloak Configuration](https://app.gitbook.com/s/u20Nc4nUTlX9s50rXkBi/resources/keycloak-configuration)
{% endcontent-ref %}

### Session initialization

The authMethod has been removed and isNewBrowserSession is to be used insted. Learn more:

{% content-ref url="https://app.gitbook.com/s/u20Nc4nUTlX9s50rXkBi/user-session-initialization" %}
[User Session Initialization](https://app.gitbook.com/s/u20Nc4nUTlX9s50rXkBi/user-session-initialization)
{% endcontent-ref %}
