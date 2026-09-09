---
icon: timer
---

# Auto Logout

Auto logout is **not** a feature you enable or disable in `oidc-spa`.\
It’s a **policy defined by your Identity Provider (IdP)**.\
What oidc-spa provides is:

* A mechanism to display a feedback overlay that warns users before they’re logged out due to inactivity.
* Ensure they never remain stuck on a stale UI where any interaction would simply redirect them to the login page.
* Monitoring of real user activity across all tabs of your application, ensuring users aren’t mistakenly marked as inactive just because they haven’t performed an action that directly contacts the IdP.

{% embed url="https://youtu.be/GeZaZIr-d68" %}
Example: Demo app with a short SSO Session Idle
{% endembed %}

***

## Understanding the Auto Logout Policy

The duration before an inactive user is logged out is **not configured in `oidc-spa,`** it’s controlled by your IdP.\
Depending on the platform, this policy might be named:

* **SSO Session Idle**
* **Idle Session Lifetime**
* **Inactivity Timeout**

When a user logs into your application, the IdP creates a **session** for that user.\
As long as this session remains active, returning to your app (with the **same browser**) automatically restores it, no new login required.

Your IdP defines how long such sessions remain active:

* **Weeks or days** → Users rarely have to log in again (e.g., Instagram, X/Twitter)
* **Minutes** → Users must log in often or may be logged out during inactivity

`oidc-spa` automatically **monitor user activity across tabs,** mouse movement, touch events, or keyboard input.\
As long as the user is active, it periodically pings the IdP to **keep the session alive**.

> 💡 **Note:**\
> IdP configuration panels often include multiple session policies.\
> For example:
>
> * **SSO Session Idle:** how long before the session expires if idle
> * **SSO Session Max / Maximum Lifetime:** total duration before forced expiration
> * **Remember Me:** may extend lifetime if selected and if not selected set session cookie to expire when the browser closes (not just the tab)

***

## Configuring Auto Logout Policy

Guides for common providers:

* [Keycloak](../providers-configuration/keycloak.md#security-sensitive-apps-banking-admin-panels-etc)
* [Auth0](../providers-configuration/auth0.md#optional-configuring-auto-logout)
* Other providers: search for:
  * “SSO Session Idle”
  * “Idle Session Lifetime”
  * “Inactivity Timeout”
  * Refresh Token TTL

***

## Verifying Auto Logout

To confirm that your IdP communicates its session policy correctly, enable debug logs:

{% tabs %}
{% tab title="Framework Agnostic" %}
{% code title="src/oidc.ts" %}
```typescript
createOidc({ 
    // ...
    debugLogs: true 
});
```
{% endcode %}
{% endtab %}

{% tab title="React" %}
{% code title="src/oidc.ts" %}
```typescript
bootstrapOidc({
    // ...
    debugLogs: true
});
```
{% endcode %}
{% endtab %}

{% tab title="Angular" %}
{% code title="src/app/app.config.ts" %}
```typescript
Oidc.provide({
  // ...
  debugLogs: true
})
```
{% endcode %}
{% endtab %}
{% endtabs %}

Then open your browser console.

If you see:

> `oidc-spa: The user will be automatically logged out after X minutes of inactivity.`

✅ You’ve successfully configured auto logout.

If instead you see:

> `oidc-spa: No refresh token, and idleSessionLifetimeInSeconds was not set, can't implement auto logout mechanism.`

It means your IdP does **not** expose this information to clients.\
In that case, you must manually specify the duration using `idleSessionLifetimeInSeconds` and keep it in sync with your IdP configuration. Keep reading for futher instructions.

***

## Auto Logout Options

{% tabs %}
{% tab title="Framework Agnostic" %}
```typescript
import { createOidc } from "oidc-spa/core";

const oidc = await createOidc({
    // ...

    // ⚠️ Read carefully:
    // Only use this if your IdP does not expose its session timeout policy.
    // (Optional) Hard-code the number of seconds of inactivity before auto logout.
    idleSessionLifetimeInSeconds: 300, // 5 minutes

    // (Optional) Where to redirect after auto logout:
    // autoLogoutParams: { redirectTo: "current page" } // Default
    // autoLogoutParams: { redirectTo: "home" }
    autoLogoutParams: {
        redirectTo: "specific url",
        get url() {
            // This let's you create a page that inform the user they have beel
            // logged out due to inactivity and display a button to come back
            // where they left off at the time of autoLogout.
            return `/activity-logout?return_url=${encodeURIComponent(location.href)}`;
        }
    }
});
```
{% endtab %}

{% tab title="React" %}
{% code title="src/oidc.ts" %}
```typescript
bootstrapOidc({
  // ...
  
  // (Optional) How long before auto logout the warning overlay should appear.
  // Default: 45 seconds
  warnUserSecondsBeforeAutoLogout: 45,
  
  // ⚠️ Read carefully:
  // Only use this if your IdP does not expose its session timeout policy.
  // (Optional) Hard-code the number of seconds of inactivity before auto logout.
  idleSessionLifetimeInSeconds: 300, // 5 minutes
    
  // (Optional) Where to redirect after auto logout:
  // autoLogoutParams: { redirectTo: "current page" } // Default
  // autoLogoutParams: { redirectTo: "home" }
  autoLogoutParams: {
      redirectTo: "specific url",
      get url() {
          // This let's you create a page that inform the user they have beel
          // logged out due to inactivity and display a button to come back
          // where they left off at the time of autoLogout.
          return `/activity-logout?return_url=${location.href}`;
      }
  }
});
```
{% endcode %}
{% endtab %}

{% tab title="Angular" %}
{% code title="src/app/app.config.ts" %}
```typescript
export const appConfig: ApplicationConfig = {
  providers: [
    // ...
    Oidc.provide({
      // ...
      
      // (Optional) How long before auto logout the overlay should appear.
      // Default: 45 seconds
      warnUserSecondsBeforeAutoLogout: 45,
      
      // ⚠️ Read carefully:
      // Only use this if your IdP does not expose its session timeout policy.
      // (Optional) Hard-code the number of seconds of inactivity before auto logout.
      idleSessionLifetimeInSeconds: 300, // 5 minutes
    
      // (Optional) Where to redirect after auto logout:
      // autoLogoutParams: { redirectTo: "current page" } // Default
      // autoLogoutParams: { redirectTo: "home" }
      autoLogoutParams: {
          redirectTo: "specific url",
          get url() {
              // This let's you create a page that inform the user they have beel
              // logged out due to inactivity and display a button to come back
              // where they left off at the time of autoLogout.
              return `/activity-logout?return_url=${location.href}`;
          }
      }
    })
  ]
}
```
{% endcode %}
{% endtab %}
{% endtabs %}

***

## Displaying a Warning Before Auto Logout

`oidc-spa` provides convenient hooks to display a **warning overlay** (or any other UI) before the user is automatically logged out.

{% tabs %}
{% tab title="Framework Agnostic" %}
Let's assume we have some utility to display an overlay modal: `showModal(message)` and `hideModal()`:

```typescript
const { unsubscribeFromAutoLogoutCountdown } =
  oidc.subscribeToAutoLogoutCountdown(({ secondsLeft }) => {
    if (secondsLeft === undefined) {
      // Countdown reset — user became active again
      hideModal();
      return;
    }
    if (secondsLeft > 60) {
      // Logout is still far away — no warning yet
      return;
    }
    showModal(`Are you still there? ${secondsLeft}s before auto logout.`);
  });
```
{% endtab %}

{% tab title="React" %}
{% code title="src/components/AutoLogoutWarningOverlay.tsx" %}
```tsx
import { useOidc } from "~/oidc";

export function AutoLogoutWarningOverlay() {
  const { autoLogoutState } = useOidc();

  if (!autoLogoutState.shouldDisplayWarning) {
    return null;
  }

  return (
    <div className="fixed inset-0 z-50 flex items-center justify-center bg-black/70 px-4 backdrop-blur">
      <div
        role="alertdialog"
        aria-live="assertive"
        aria-modal="true"
        className="w-full max-w-sm rounded-2xl border border-slate-800 bg-slate-900 p-6 text-center shadow-xl shadow-black/30"
      >
        <p className="text-sm font-medium text-slate-400">
          Are you still there?
        </p>
        <p className="mt-2 text-lg font-semibold text-white">
          You will be logged out in {autoLogoutState.secondsLeftBeforeAutoLogout}s
        </p>
      </div>
    </div>
  );
}
```
{% endcode %}

Then mount it near the root of your app:

<pre class="language-tsx" data-title="src/App.tsx"><code class="lang-tsx"><strong>import { AutoLogoutWarningOverlay } from "./components/AutoLogoutWarningOverlay";
</strong>
export function App() {
  return (
    &#x3C;div>
      &#x3C;Header />
      &#x3C;main>{/* ... */}&#x3C;/main>
<strong>      &#x3C;AutoLogoutWarningOverlay />
</strong>    &#x3C;/div>
  );
}
</code></pre>
{% endtab %}

{% tab title="Angular" %}
{% code title="src/app/app.html" %}
```html
<header>...</header>

<router-outlet />

@if (oidc.$secondsLeftBeforeAutoLogout()) {
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
    zIndex: 1000
  }">
    <div>
      <p>Are you still there?</p>
      <p>You will be logged out in {{ oidc.$secondsLeftBeforeAutoLogout() }}</p>
    </div>
  </div>
}
```
{% endcode %}
{% endtab %}
{% endtabs %}

***
