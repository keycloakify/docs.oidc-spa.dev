---
description: >-
  Automatically logging out your user after a set period of inactivity on your
  app (they dont move the mouse or press any key on the keyboard for a while)
---

# ⏲️ Auto Logout

### Configuring auto logout policy

This is a policy that is enforced on the identity server.

The auto logout is defined by the lifespan of the refresh token.

For example, if you're using Keycloak and you want you want an auto disconnect after 10 minutes of inactivity you would set the SSO Session Idle to 10 minutes. See [Keycloak configuration guide](../resources/usage-with-keycloak.md).

If you can't configure your identity provider you can still enforce auto logout like so:

{% tabs %}
{% tab title="Vanilla API" %}
```typescript
import { createOidc } from "oidc-spa";

const oidc = await createOidc({
  // ...
  __unsafe_ssoSessionIdleSeconds: 10 * 60 // 10 minutes
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
    __unsafe_ssoSessionIdleSeconds: 10 * 60 // Ten minutes
    //autoLogoutParams: { redirectTo: "current page" } // Default
    //autoLogoutParams: { redirectTo: "home" }
    //autoLogoutParams: { redirectTo: "specific url", url: "/a-page" }
});
```
{% endtab %}
{% endtabs %}

Note that this parameter is marked as unsafe because what happens if the user closes the tab? He will be able to return a while back and still be logged in. oidc-spa can't enforce a security policy when it's not running. Only the identity server can.

### Displaying a coutdown timer before auto logout

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
Example implementation of a 60 seconds countdown before autologout.
{% endembed %}
{% endtab %}
{% endtabs %}
