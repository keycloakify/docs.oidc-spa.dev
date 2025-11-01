---
icon: timer
---

# Auto Logout

The Auto Logout is not a feature that you enable or disable. It's a policty of your IdP. oidc-spa simply let you implement a feedback overlay that will warn your user when they are about to be loged out due to inactivity. Example:

{% embed url="https://youtu.be/GeZaZIr-d68" %}
The demo app with a short SSO Session Idle
{% endembed %}

## Understanding the Auto Logount Policy

Again, "after how long an inactive user should be automatically logged out" is not a parameter that you pass to oidc-spa. It's a policty you configure in the administration panel of your IdP that is usually refered to as "SSO Session Idle" or "Idle Session Lifetime" or "Inactivity Timeout".

When a user logs into your application the IdP will create a session for this user.  \
While this session is active, whenever the user return to your app [with the same browser](#user-content-fn-1)[^1] they would not have to go through the login process again their session will be autmatically restored, they will be automatically logged in.

Now your IdP always have a policy that dictate for how long a session should be kept alive.

If it's in orders of weeks the user will almost never have to login again to you app as long as they access it with the same browser.  Typical examples: Instagram, X.

If it's in an order of minutes, users will have to login agin every time they access your app and even can be loogged out automatically if they are being inactive for too long. &#x20;

Note that oidc-spa tracks the activity of the user across the different tabs of your app. And by activity we mean anty movment of the mouse, touch event or keyboard input. While the user is active oidc-spa will make sure to "ping" the auth server periodically to keep the session alive.

NOTE: There are a lot of satelite policy that can make the thing feel overwhelming in your admin pannel. On your IdP "SSO Session Idle" is often paired with an other parameter "SSO Session Max" or "Maximum Liftime" that define after how long the session should be discared even if the user is active. Some provider like Keycloak also provide "Remember Me" checkbox that changes this policy depending if the user has clicked that or not on top of sessing cookies that disapers when the browser is closed (not the tab).

## Configuring auto logout policy

Guide on how to configure it by provider:

* [Keycloak](providers-configuration/keycloak.md#security-sensitive-apps-banking-admin-panels-etc)
* [Auth0](providers-configuration/auth0.md#optional-configuring-auto-logout)
* Other: Search for "SSO Session Idle" or "Idle Session Lifetime" or "Inactivity Timeout"

## Making sure that Auto Logout is enabled

To check if you have correctly set the value you can set `debugLogs: true`, and open de console. If you see:

<figure><img src=".gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

> oidc-spa: The user will be automatically logged out after X minutes of inactivity.

You have sussesfully set auto logout!

If on the other hand you see:&#x20;

<figure><img src=".gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

> oidc-spa: No refresh token, and idleSessionLifetimeInSeconds was not set, can't implement auto logout mechanism.

Then it means that you IdP does not propagate this information and you will have to specify this value explicitly with idleSessionLifetimeInSeconds and keep it in sync this value and your IdP Config.

## Auto Logout Options



{% tabs %}
{% tab title="Framwork Agnositc" %}
```typescript
import { createOidc } from "oidc-spa";

const oidc = await createOidc({
  // ...
  
  // ‼️ WARNING ‼️ Read carfully what's above.
  // This is only as last resort if you use an IdP that do not comunicate it's
  // policy regarding auto logout to it's client.
  // (OPTIONAL) Hard code after how many seconds of inactivity the user should
  // be auto logged out.
  idleSessionLifetimeInSeconds: 300 // 5 minutes
    
  // (OPTIONAL) Where to redirect when the autoLogout kiks in: 
  autoLogoutParams: { redirectTo: "current page" } // Default, higly recomended
  //autoLogoutParams: { redirectTo: "home" }
  //autoLogoutParams: { redirectTo: "specific url", url: "/a-page" }
  
});
```
{% endtab %}

{% tab title="React" %}
{% code title="src/oidc.ts" %}
```typescript
bootstrapOidc({
  // ...
  
  // (OPTIONAL) This is for configuring the how long before auto logout
  // the overlay that you might have implemented should be displayed.
  // See next section.
  // Default: 45 seconds
  warnUserSecondsBeforeAutoLogout: 45
  
  // ‼️ WARNING ‼️ Read carfully what's above.
  // This is only as last resort if you use an IdP that do not comunicate it's
  // policy regarding auto logout to it's client.
  // (OPTIONAL) Hard code after how many seconds of inactivity the user should
  // be auto logged out.
  idleSessionLifetimeInSeconds: 300 // 5 minutes
    
  // (OPTIONAL) Where to redirect when the autoLogout kiks in: 
  autoLogoutParams: { redirectTo: "current page" } // Default, higly recomended
  //autoLogoutParams: { redirectTo: "home" }
  //autoLogoutParams: { redirectTo: "specific url", url: "/a-page" }
    
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
      
      // (OPTIONAL) This is for configuring the how long before auto logout
      // the overlay that you might have implemented should be displayed.
      // See next section.
      // Default: 45 seconds
      warnUserSecondsBeforeAutoLogout: 45
      
      // ‼️ WARNING ‼️ Read carfully what's above.
      // This is only as last resort if you use an IdP that do not comunicate it's
      // policy regarding auto logout to it's client.
      idleSessionLifetimeInSeconds: 300 // 5 minutes
    
      //autoLogoutParams: { redirectTo: "current page" } // Default, recommended
      //autoLogoutParams: { redirectTo: "home" }
      //autoLogoutParams: { redirectTo: "specific url", url: "/a-page" }
      
    })
  ]
};
```
{% endcode %}
{% endtab %}
{% endtabs %}

## Warning the user before auto logout

oidc-spa make it easy to display a warning overlay (or other UI) to warn inactive user about the fact that they are about to be logged out if they don't interact with the app. &#x20;

{% tabs %}
{% tab title="Framework Agnostic" %}
```typescript
const { 
  unsubscribeFromAutoLogoutCountdown 
} = oidc.subscribeToAutoLogoutCountdown(({ secondsLeft }) => {
    if( secondsLeft === undefined ){
      // Countdown reset, the user interacted with the app (moved the mouse,
      // typed something on the keyboard...)
      hideModal();
      return;
    }
    if( secondsLeft > 60 ){
      // Auto logout is in a long time, no need to warn.
      return;
    }
    showModal(`Are you still here? ${secondsLeft} before auto logout`);
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

        // By default we start displaying the warning 45 seconds before
        // auto logout. If you 
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

Then mount this component in your `__root.tsx`:

<pre class="language-tsx" data-title="src/routes/__root.tsx"><code class="lang-tsx">import { HeadContent, Scripts, createRootRoute } from "@tanstack/react-router";

import Header from "@/components/Header";
<strong>import { AutoLogoutWarningOverlay } from "@/components/AutoLogoutWarningOverlay";
</strong>
export const Route = createRootRoute({
    head: () => ({
        meta: [ /* ... */ ],
        links: [ /* ... */]
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
                &#x3C;main>
                    {children}
                &#x3C;/main>
<strong>                &#x3C;AutoLogoutWarningOverlay />
</strong>                &#x3C;Scripts />
            &#x3C;/body>
        &#x3C;/html>
    );
}
</code></pre>
{% endtab %}

{% tab title="React SPAs" %}
You can have a component like this one mounted at all time:

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

Then you can mount this component close to the root, for example:&#x20;

<pre class="language-tsx" data-title="src/App.tsx"><code class="lang-tsx"><strong>import { AutoLogoutWarningOverlay } from "./components/AutoLogoutWarningOverlay";
</strong>
export function App() {
    return (
        &#x3C;div>
            &#x3C;Header />
            &#x3C;main>{/*...*/}&#x3C;/main>
<strong>            &#x3C;AutoLogoutWarningOverlay />
</strong>        &#x3C;/div>
    );
}

</code></pre>
{% endtab %}

{% tab title="Angular" %}
{% code title="src/app/app.html" %}
```html
<header>...</header>

<router-outlet />

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
```
{% endcode %}
{% endtab %}
{% endtabs %}

[^1]: 
