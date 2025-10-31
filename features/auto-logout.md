---
description: >-
  Automatically logging out your user after a set period of inactivity on your
  app (they dont move the mouse or press any key on the keyboard for a while)
icon: timer
---

# Auto Logout

### Configuring auto logout policy

**Important to understand**: This is a policy that is enforced on the identity server. Not in the application code!

In OIDC provider, it is usually referred to as **Idle Session Lifetime**, these values define how long an inactive session should be kept in the records of the server.

Guide on how to configure it:

* [Keycloak](../providers-configuration/keycloak.md#security-sensitive-apps-banking-admin-panels-etc)
* [Auth0](../providers-configuration/auth0.md#optional-configuring-auto-logout)

[If your OIDC provider issues a Refresh Token and if this refresh token is a JWT](#user-content-fn-1)[^1] you don't need to configure anything at the app level. Otherwise you need to explicitly set the `idleSessionLifetimeInSeconds` so it matches with how you have configured your server.

{% tabs %}
{% tab title="Framwork Agnositc" %}
```typescript
import { createOidc } from "oidc-spa";

const oidc = await createOidc({
  // ...
  // ‼️ WARNING ‼️ Read carfully what's above.
  // Use idleSessionLifetimeInSeconds if and only if you are using an auth server
  // that do not let you configure this policy! (e.g. if you're using Keycloak don't use this param) 
  idleSessionLifetimeInSeconds: 300 // 5 minutes
    
  //autoLogoutParams: { redirectTo: "current page" } // Default
  //autoLogoutParams: { redirectTo: "home" }
  //autoLogoutParams: { redirectTo: "specific url", url: "/a-page" }
});
```
{% endtab %}

{% tab title="React SPA" %}
```typescript
import { createReactOidc } from "oidc-spa/react";

export const {
    OidcProvider,
    useOidc
} = createReactOidc({
  // ...
  // ‼️ WARNING ‼️ Read carfully what's above.
  // Use idleSessionLifetimeInSeconds if and only if you are using an auth server
  // that do not let you configure this policy! (e.g. if you're using Keycloak don't use this param) 
  idleSessionLifetimeInSeconds: 300 // 5 minutes
    
  //autoLogoutParams: { redirectTo: "current page" } // Default
  //autoLogoutParams: { redirectTo: "home" }
  //autoLogoutParams: { redirectTo: "specific url", url: "/a-page" }
});
```
{% endtab %}

{% tab title="TanStack Start" %}
Mount this component in your \_\_root.tsx

```typescript
import { createOidcComponent } from "@/oidc";

/** See: https://docs.oidc-spa.dev/auto-logout */
export const AutoLogoutWarningOverlay = createOidcComponent({
    component: () => {
        const { autoLogoutState } = AutoLogoutWarningOverlay.useOidc();

        if (!autoLogoutState.shouldDisplayWarning) {
            return null;
        }

        return (
            <div
                // Full screen overlay, blurred background
                style={{
                    position: "fixed",
                    top: 0,
                    left: 0,
                    right: 0,
                    bottom: 0,
                    backgroundColor: "rgba(0,0,0,0.5)",
                    backdropFilter: "blur(10px)",
                    display: "flex",
                    justifyContent: "center",
                    alignItems: "center",
                    zIndex: 1000,
                    color: "white"
                }}
            >
                <div>
                    <p>Are you still there?</p>
                    <p>You will be logged out in {autoLogoutState.secondsLeftBeforeAutoLogout}</p>
                    {/* NOTE: You can configure how long before autoLogout we start displaying
                        this warning by providing `startCountdownSecondsBeforeAutoLogout` 
                        to bootstrapOidc()
                    */}
                </div>
            </div>
        );
    }
});
```
{% endtab %}

{% tab title="Angular" %}
See [example](https://github.com/keycloakify/oidc-spa/blob/main/examples/angular/src/app/app.html).

```html
@if (oidc.$secondsLeftBeforeAutoLogout() ) {
<!-- Full screen overlay, blurred background -->
<div [style]="{
      position: 'fixed',
      top: 0,
      left: 0,
      right: 0,
      bottom: 0,
      backgroundColor: 'rgba(0,0,0,0.5)',
      backdropFilter: 'blur(10px)',
      display: 'flex',
      justifyContent: 'center',
      alignItems: 'center',
      zIndex: 1000,
    }">
      <div>
            <p>Are you still there?</p>
            <p>You will be logged out in {{ oidc.$secondsLeftBeforeAutoLogout() }}</p>
      </div>
</div>
}
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
You can have a component like this one mounted at all time:

```tsx
import { useOidc } from "../oidc";

export function AutoLogoutWarningOverlay() {
    const { useAutoLogoutWarningCountdown } = useOidc();
    const { secondsLeft } = useAutoLogoutWarningCountdown({ 
        // How many seconds before auto logout do we start
        // displaying the overlay.
        warningDurationSeconds: 45 
    });

    if (secondsLeft === undefined) {
        return null;
    }

    return (
        <div
            // Full screen overlay, blurred background
            style={{
                position: "fixed",
                top: 0,
                left: 0,
                right: 0,
                bottom: 0,
                backgroundColor: "rgba(0,0,0,0.5)",
                backdropFilter: "blur(10px)",
                display: "flex",
                justifyContent: "center",
                alignItems: "center",
                zIndex: 1000
            }}
        >
            <div>
                <p>Are you still there?</p>
                <p>You will be logged out in {secondsLeft}</p>
            </div>
        </div>
    );
}
```
{% endtab %}
{% endtabs %}

[^1]: If you're not sure use `createOidc({ debugLogs: true })` and see what's printed in the console after you log in.\
    \
    For reference, if you are using Keycloak, you do not need to provide the `idleSessionLifetimeInSeconds` parameter to oidc-spa, with Auth0, you do.
