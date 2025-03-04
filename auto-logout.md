---
icon: timer
description: >-
  Automatically logging out your user after a set period of inactivity on your
  app (they dont move the mouse or press any key on the keyboard for a while)
---

# Auto Logout

### Configuring auto logout policy

**Important to understand**: This is a policy that is enforced on the identity server. Not in the application code!

In OIDC provider, it is usually referred to as **Idle Session Lifetime**, these values define how long an inactive session should be kept in the records of the server.

Guide on how to configure it:

* [Keycloak](providers-configuration/keycloak.md#security-sensitive-apps-banking-admin-panels-etc)
* [Auth0](providers-configuration/auth0.md#optional-configuring-auto-logout)

[If your OIDC provider issues a Refresh Token and if this refresh token is a JWT](#user-content-fn-1)[^1] you don't need to configure anything at the app level. Otherwise you need to explicitly set the `idleSessionLifetimeInSeconds` so it matches with how you have configured your server.

{% tabs %}
{% tab title="Vanilla API" %}
```typescript
import { createOidc } from "oidc-spa";

const oidc = await createOidc({
  // ...
  idleSessionLifetimeInSeconds: 300 // 5 minutes
  //autoLogoutParams: { redirectTo: "current page" } // Default
  //autoLogoutParams: { redirectTo: "home" }
  //autoLogoutParams: { redirectTo: "specific url", url: "/a-page" }
});
```
{% endtab %}

{% tab title="React API" %}
```typescript
import { createReactOidc } from "oidc-spa/react";

export const {
    OidcProvider,
    useOidc
} = createReactOidc({
    // ...
    __unsafe_ssoSessionIdleSeconds: 300 // 5 minuts
    //autoLogoutParams: { redirectTo: "current page" } // Default
    //autoLogoutParams: { redirectTo: "home" }
    //autoLogoutParams: { redirectTo: "specific url", url: "/a-page" }
});
```
{% endtab %}
{% endtabs %}

### Displaying a countdown timer before auto logout

{% embed url="https://youtu.be/GeZaZIr-d68" %}
The demo app with a short SSO Session Idle
{% endembed %}

{% tabs %}
{% tab title="Vanilla API" %}
```typescript
const { unsubscribeFromAutoLogoutCountdown } = oidc.subscribeToAutoLogoutCountdown(
  ({ secondsLeft }) => {
    if( secondsLeft === undefined ){
      console.log("Countdown reset, the user moved");
      return;
    }
    if( secondsLeft > 60 ){
      return;
    }
    console.log(`${secondsLeft} before auto logout`)
  }
);
```
{% endtab %}

{% tab title="React API" %}
{% embed url="https://github.com/keycloakify/oidc-spa/blob/main/examples/tanstack-router/src/router/AutoLogoutCountdown.tsx" %}
Example implementation of a 60 seconds countdown before auto logout.
{% endembed %}
{% endtab %}
{% endtabs %}

[^1]: If you're not sure use `createOidc({ debugLogs: true })` and see what's printed in the console after you log in.\
    \
    For reference, if you are using Keycloak, you do not need to provide the `idleSessionLifetimeInSeconds` parameter to oidc-spa, with Auth0, you do.
