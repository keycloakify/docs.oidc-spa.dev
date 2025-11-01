---
icon: timer
---

# Auto Logout

Auto logout is **not** a feature you enable or disable in `oidc-spa`.  
It’s a **policy defined by your Identity Provider (IdP)**.  
What `oidc-spa` provides is a way to **display a feedback overlay** that warns users before they are logged out due to inactivity.

{% embed url="https://youtu.be/GeZaZIr-d68" %}
Example: Demo app with a short SSO Session Idle
{% endembed %}

---

## Understanding the Auto Logout Policy

The duration before an inactive user is logged out is **not configured in `oidc-spa`** — it’s controlled by your IdP.  
Depending on the platform, this policy might be named:

* **SSO Session Idle**
* **Idle Session Lifetime**
* **Inactivity Timeout**

When a user logs into your application, the IdP creates a **session** for that user.  
As long as this session remains active, returning to your app (with the **same browser**) automatically restores it — no new login required.

Your IdP defines how long such sessions remain active:

* **Weeks or days** → Users rarely have to log in again (e.g., Instagram, X/Twitter)
* **Minutes** → Users must log in often or may be logged out during inactivity

`oidc-spa` automatically **tracks user activity across tabs** — mouse movement, touch events, or keyboard input.  
As long as the user is active, it periodically pings the IdP to **keep the session alive**.

> 💡 **Note:**  
> IdP configuration panels often include multiple session policies.  
> For example:
> * **SSO Session Idle** — how long before the session expires if idle  
> * **SSO Session Max / Maximum Lifetime** — total duration before forced expiration  
> * **Remember Me** — may extend lifetime if selected  
> * **Session cookies** — expire when the browser closes (not just the tab)

---

## Configuring Auto Logout Policy

Guides for common providers:

* [Keycloak](providers-configuration/keycloak.md#security-sensitive-apps-banking-admin-panels-etc)
* [Auth0](providers-configuration/auth0.md#optional-configuring-auto-logout)
* Other providers — search for:  
  * “SSO Session Idle”  
  * “Idle Session Lifetime”  
  * “Inactivity Timeout”

---

## Verifying Auto Logout

To confirm that your IdP communicates its session policy correctly, enable debug logs:

```ts
debugLogs: true
```

Then open your browser console.

If you see:

> `oidc-spa: The user will be automatically logged out after X minutes of inactivity.`

✅ You’ve successfully configured auto logout.

If instead you see:

> `oidc-spa: No refresh token, and idleSessionLifetimeInSeconds was not set, can't implement auto logout mechanism.`

It means your IdP does **not** expose this information to clients.  
In that case, you must manually specify the duration using `idleSessionLifetimeInSeconds` and keep it in sync with your IdP configuration.

---

## Auto Logout Options

{% tabs %}
{% tab title="Framework Agnostic" %}
```typescript
import { createOidc } from "oidc-spa";

const oidc = await createOidc({
  // ...
  
  // ⚠️ Read carefully:
  // Only use this if your IdP does not expose its session timeout policy.
  // Hard-code the number of seconds of inactivity before auto logout.
  idleSessionLifetimeInSeconds: 300, // 5 minutes
    
  // (Optional) Where to redirect after auto logout:
  autoLogoutParams: { redirectTo: "current page" } // Default (recommended)
  // autoLogoutParams: { redirectTo: "home" }
  // autoLogoutParams: { redirectTo: "specific url", url: "/a-page" }
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
  
  // ⚠️ Only use this if your IdP does not expose its policy.
  // Manually specify the idle session lifetime.
  idleSessionLifetimeInSeconds: 300, // 5 minutes
    
  // (Optional) Redirect behavior after auto logout:
  autoLogoutParams: { redirectTo: "current page" } // Default (recommended)
  // autoLogoutParams: { redirectTo: "home" }
  // autoLogoutParams: { redirectTo: "specific url", url: "/a-page" }
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
      
      // ⚠️ Only use this if your IdP does not expose its policy.
      idleSessionLifetimeInSeconds: 300, // 5 minutes
    
      // autoLogoutParams: { redirectTo: "current page" } // Default (recommended)
      // autoLogoutParams: { redirectTo: "home" }
      // autoLogoutParams: { redirectTo: "specific url", url: "/a-page" }
    })
  ]
};
```
{% endcode %}
{% endtab %}
{% endtabs %}

---

## Displaying a Warning Before Auto Logout

`oidc-spa` provides convenient hooks to display a **warning overlay** (or any other UI) before the user is automatically logged out.

{% tabs %}
{% tab title="Framework Agnostic" %}
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

{% tab title="TanStack Start" %}
{% code title="src/components/AutoLogoutWarningOverlay.tsx" %}
```tsx
import { createOidcComponent } from "@/oidc";

export const AutoLogoutWarningOverlay = createOidcComponent({
  component: () => {
    const { autoLogoutState } = AutoLogoutWarningOverlay.useOidc();

    // Default: starts displaying 45 seconds before auto logout
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
});
```
{% endcode %}

Then mount it in your root layout:

<pre class="language-tsx" data-title="src/routes/__root.tsx"><code class="lang-tsx">import { HeadContent, Scripts, createRootRoute } from "@tanstack/react-router";
import Header from "@/components/Header";
<strong>import { AutoLogoutWarningOverlay } from "@/components/AutoLogoutWarningOverlay";
</strong>
export const Route = createRootRoute({
  head: () => ({
    meta: [/* ... */],
    links: [/* ... */]
  }),
  shellComponent: RootDocument
});

function RootDocument({ children }: { children: React.ReactNode }) {
  return (
    &#x3C;html lang="en">
      &#x3C;head>
        &#x3C;HeadContent />
      &#x3C;/head>
      &#x3C;body>
        &#x3C;Header />
        &#x3C;main>{children}&#x3C;/main>
<strong>        &#x3C;AutoLogoutWarningOverlay />
</strong>        &#x3C;Scripts />
      &#x3C;/body>
    &#x3C;/html>
  );
}
</code></pre>
{% endtab %}

{% tab title="React SPAs" %}
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

---

[^1]: Applies when using the same browser profile and storage context.  
[^2]: Pass `debugLogs: true` to `createOidc({})` or `bootstrapOidc({})`.
