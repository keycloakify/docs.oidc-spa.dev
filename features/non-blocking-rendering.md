---
icon: gauge-max
---

# Non Blocking Rendering

This section explains how to configure your application so it can begin rendering before the user’s authentication state is fully determined.

With this setup, the initial UI appears immediately, and authentication-aware components are rendered a moment later once the auth state is resolved. The result looks like this video:

{% embed url="https://www.youtube.com/watch?v=t1qfU_GeTM4" %}

{% tabs %}
{% tab title="React SPAs" %}
### Default: blocking rendering (simplest)

<pre class="language-tsx" data-title="src/main.tsx"><code class="lang-tsx">import React from "react";
import ReactDOM from "react-dom/client";
import { App } from "./App";
<strong>import { OidcInitializationGate } from "~/oidc";
</strong>

ReactDOM.createRoot(document.getElementById("root")!).render(
    &#x3C;React.StrictMode>
<strong>        &#x3C;OidcInitializationGate>
</strong>            &#x3C;App />
<strong>        &#x3C;/OidcInitializationGate>
</strong>    &#x3C;/React.StrictMode>
);
</code></pre>

By default, this setup **defers rendering your entire app** until `bootstrapOidc()` has resolved, in other words, until oidc-spa has contacted your IdP and determined whether the user currently has an active session.

This is often the **simplest and safest** choice:

* You don’t have to think about whether the auth state has settled.
* There’s no risk of layout shifts.

You just need to make sure to at least [set the background color early to avoid white flashes](https://github.com/keycloakify/oidc-spa/blob/c39b0fb70a576e62602d99e9ef86211532de1e35/examples/react-router-declarative/src/index.css#L12).

***

### Faster first paint: non-blocking rendering

However, for **optimal performance**, you can start rendering _before_ the authentication state is resolved, letting the page appear instantly, while auth-aware components hydrate a few milliseconds later.

For example:

{% embed url="https://youtu.be/t1qfU_GeTM4?si=xrbRvl9dJQS9xccJ" %}

In this short demo, the homepage renders immediately, and components depending on authentication appear shortly after the session check completes.

You can achieve this simply by moving `<OidcInitializationGate />` closer to the components that call `useOidc()`:

First, you need to remove the root OidcInitializationGate:

{% code title="src/main.tsx" %}
```diff
 import React from "react";
 import ReactDOM from "react-dom/client";
 import { BrowserRouter } from "react-router";
 import { App } from "./App";
-import { OidcInitializationGate } from "~/oidc";
 import "./index.css";

 ReactDOM.createRoot(document.getElementById("root")!).render(
     <React.StrictMode>
-         <OidcInitializationGate>
             <BrowserRouter>
                 <App />
             </BrowserRouter>
-         </OidcInitializationGate>
     </React.StrictMode>
 );
```
{% endcode %}

Then wrap all the components that call the useOidc() hook without assertion, into `<OidcInitializationGate />` or `<Suspense />`:

{% hint style="warning" %}
Don't forget `<AutoLogoutWarningOverlay />`! If you forget to wrap a single component that call useOidc(), you're all app will suspend.
{% endhint %}

{% hint style="success" %}
The components that call useOidc({ assert: "..." }) do **not** need to be wrapped into `OidcInitializationGate`! If you are able to make an assertion, the auth state has been established already and those calls will never suspend!
{% endhint %}

<pre class="language-tsx" data-title="src/components/Header.tsx"><code class="lang-tsx">import { Suspense } from "react";
import { 
    useOidc, 
<strong>    OidcInitializationGate 
</strong>} from "~/oidc";

export function Header() {
    return (
        &#x3C;header>
            {/* ... */}
<strong>            &#x3C;OidcInitializationGate fallback={&#x3C;Spinner />}>
</strong>                &#x3C;AuthButtons />
<strong>            &#x3C;/OidcInitializationGate>
</strong>        &#x3C;/header>
    );
}

function AuthButtons() {
    const { isUserLoggedIn } = useOidc();

    return (
        &#x3C;div className="animate-fade-in">
            {isUserLoggedIn ? &#x3C;LoggedInAuthButtons /> : &#x3C;NotLoggedInAuthButtons />}
        &#x3C;/div>
    );
}
</code></pre>

***

### Using React’s built-in Suspense

**If your app is a pure SPA (no SSR)**, you can use React’s built-in `<Suspense />` interchangably with `<OidcInitializationGate />`.\
This is often even better, as it lets you define a unified fallback for all your app’s asynchronous operations.

***

### `withLoginEnforced()`&#x20;

Components wrapped in `withLoginEnforced()` behave like a component that calls `useOidc()`. That's all you need to know. \
\
**Pro tip:** In a pure SPA (no SSR), you can use `<Suspense />` instead of `<OidcInitializationGate />` to wrap your routes and have a single `<Spinner />` both for the page lazy loading and as fallback while the oidc is initializing.

{% code title="src/pages/Protected.tsx" %}
```tsx
import { withLoginEnforced } from "~/oidc";

const Protected = withLoginEnforced(() => {
    return <div>{/* ... */}</div>;
});

export default Protected;
```
{% endcode %}

Example:

<pre class="language-tsx" data-title="src/App.tsx"><code class="lang-tsx">import { lazy, Suspense } from "react";
import { Navigate, Route, Routes } from "react-router";
import { AutoLogoutWarningOverlay } from "./components/AutoLogoutWarningOverlay";
import { Header } from "./components/Header";
import { Home } from "./pages/Home";
import { OidcInitializationGate } from "~/oidc";
const Protected = lazy(() => import("./pages/Protected"));
const AdminOnly = lazy(() => import("./pages/AdminOnly"));

export function App() {
    return (
        &#x3C;>
            &#x3C;Header />
            &#x3C;main>
<strong>                &#x3C;Suspense fallback={&#x3C;Spinner />}>
</strong>                    &#x3C;Routes>
                        &#x3C;Route index element={&#x3C;Home />} />
                        &#x3C;Route path="protected" element={&#x3C;Protected />} />
                        &#x3C;Route path="admin-only" element={&#x3C;AdminOnly />} />
                        &#x3C;Route path="*" element={&#x3C;Navigate to="/" replace />} />
                    &#x3C;/Routes>
<strong>                &#x3C;/Suspense>
</strong>            &#x3C;/main>
            &#x3C;OidcInitializationGate>
                &#x3C;AutoLogoutWarningOverlay />
            &#x3C;/OidcInitializationGate>
        &#x3C;/>
    );
}
</code></pre>

***

### TL;DR

* `<OidcInitializationGate />` at the root: **simpler mental model**, no layout shift.
* `<Suspense />` or `<OidcInitializationGate />` near `useOidc()` calls: **faster perceived load**, better user experience.
* Components using `useOidc({ assert: "..." })` do **not** need to be wrapped, they will never suspend.
* If you use `withLoginEnforced()` it need to be wrapped as well.
* Don't forget to wrap `AutoLogoutWarningOverlay`

***

_(In modern browsers, session restoration typically takes under 300 ms, so even full gating often feels instant.)_
{% endtab %}

{% tab title="Angular" %}
### Default: blocking rendering (simplest and safest)

When using the `oidc-spa/angular` adapter, the recommended default is to **let bootstrap wait for OIDC**.\
You do this by using your `Oidc` service and **not** opting out of provider waiting (the default).

**app.config.ts**

<pre class="language-ts"><code class="lang-ts">import { ApplicationConfig } from '@angular/core';
import { provideRouter } from '@angular/router';
import { routes } from './app.routes';
import { Oidc } from './services/oidc.service';

export const appConfig: ApplicationConfig = {
  providers: [
<strong>    // This will NOT resolve until bootstrapOidc() completes.
</strong>    Oidc.provide({
      // ...
    }),
    provideRouter(routes),
  ],
};
</code></pre>

**Oidc service (simple example)**

```ts
import { Injectable } from '@angular/core';
import { AbstractOidcService } from 'oidc-spa/angular';

export type DecodedIdToken = {
  name: string;
  realm_access?: { roles: string[] };
};

@Injectable({ providedIn: 'root' })
export class Oidc extends AbstractOidcService<DecodedIdToken> {
  // providerAwaitsInitialization defaults to true
}
```

With this setup, Angular only renders once `bootstrapOidc()` has completed (the IdP has been contacted and the session state is known).

**Why this is nice**

* You do not think about “is OIDC ready”.
* No layout shifts.
* Tests and SSR behave predictably. (NOTE: SSR in Angular not tested yet)

***

### Faster first paint: non-blocking rendering

For optimal performance, you can start rendering **before** the authentication state is fully resolved, so the page appears instantly and OIDC-aware parts “hydrate” moments later.

Example of what it can look in action:

{% embed url="https://youtu.be/t1qfU_GeTM4" %}

Enable this by opting out of provider waiting in your `Oidc` service:

<pre class="language-ts"><code class="lang-ts">// examples/angular-kitchensink/src/app/services/oidc.service.ts
import { Injectable } from '@angular/core';
import { AbstractOidcService } from 'oidc-spa/angular';

@Injectable({ providedIn: 'root' })
export class Oidc extends AbstractOidcService {
  // The provider no longer blocks Angular bootstrap
<strong>  override providerAwaitsInitialization = false;
</strong>
  // ...
}
</code></pre>

**Important:** Once you do this, **you** are responsible for placing “init boundaries” in templates, so parts of the UI that need OIDC only render once it is ready.

#### Gate OIDC-aware UI with `@defer`

Use Angular’s built-in `@defer` with a `@placeholder` for instant paint:

<pre class="language-html"><code class="lang-html">&#x3C;!-- examples/angular-kitchensink/src/app/app.html -->
&#x3C;header>
  &#x3C;span>OIDC-SPA + Angular (Kitchen Sink)&#x3C;/span>

<strong>  @defer (when oidc.prInitialized | async) {
</strong>    &#x3C;!-- Safe to read OIDC values here -->
    @if (oidc.isUserLoggedIn) {
      &#x3C;div>
        &#x3C;span>Hello {{ oidc.$decodedIdToken().name }}&#x3C;/span>
        &#x26;nbsp; &#x3C;button (click)="oidc.logout({ redirectTo: 'home' })">Logout&#x3C;/button>
      &#x3C;/div>
    } @else {
      &#x3C;div>
        &#x3C;button (click)="oidc.login()">Login&#x3C;/button>
        &#x3C;button (click)="
          oidc.login({
            transformUrlBeforeRedirect: keycloakUtils.transformUrlBeforeRedirectForRegister,
          })
        ">
          Register
        &#x3C;/button>
      &#x3C;/div>
    }
<strong>  } @placeholder {
</strong><strong>    &#x3C;span style="line-height: 1.35;">Initializing OIDC...&#x3C;/span>
</strong><strong>  }
</strong>&#x3C;/header>
</code></pre>

Anywhere you read things like `oidc.isUserLoggedIn`, `oidc.$decodedIdToken()`, or values derived from `issuerUri`, put them behind a `@defer (when oidc.prInitialized | async)` (or otherwise guard them) to avoid runtime errors during the brief initialization window.

#### Access helpers lazily to avoid crashes

Because the component can be constructed before OIDC is initialized, compute helpers like `keycloakUtils` **lazily**:

<pre class="language-ts" data-title="src/app/app.ts"><code class="lang-ts">import { Component, inject } from '@angular/core';
import { Oidc } from './services/oidc.service';
import { createKeycloakUtils } from 'oidc-spa/keycloak';

@Component({
  selector: 'app-root',
  templateUrl: './app.html',
  imports: [],
})
export class App {
  oidc = inject(Oidc);

  // Use a getter so we read issuerUri only after init
<strong>  get keycloakUtils() {
</strong><strong>    return createKeycloakUtils({ issuerUri: this.oidc.issuerUri });
</strong><strong>  }
</strong>
  // Example: drive an "Admin only" link state
  get canShowAdminLink(): boolean {
    if (!this.oidc.isUserLoggedIn) return true;
    const roles = this.oidc.$decodedIdToken().realm_access?.roles ?? [];
    return roles.includes('admin');
  }
}
</code></pre>

***

### TL;DR

* **Blocking at bootstrap (default):** `Oidc.provide()` waits for `bootstrapOidc()` before Angular renders. Easiest mental model. No layout shift. Tests and SSR are straightforward.
* **Non-blocking:** set `override providerAwaitsInitialization = false` in your `Oidc` service. Then:
  * Gate auth-aware UI with `@defer (when oidc.prInitialized | async) { ... } @placeholder { ... }`.
  * Access helpers like `keycloakUtils` via a **getter** so you do not touch `issuerUri` before init.
  * Guard overlays or any code that reads OIDC state.
* Choose based on the UX you want. Both modes are supported.

_(In modern browsers, session restoration usually completes in under \~300 ms, so even full gating often feels instant.)_
{% endtab %}

{% tab title="TanStack Start" %}
In TanStack Start, non-blocking rendering is the default, since it's required for server rendering.\
However, if you find the layout shift caused by auth-aware components appearing _after_ hydration annoying to handle, you can easily delay rendering your app until the OIDC initialization process has completed:

<pre class="language-tsx" data-title="src/routes/__root.tsx"><code class="lang-tsx">import { HeadContent, Scripts, createRootRoute } from "@tanstack/react-router";
import Header from "@/components/Header";
import { AutoLogoutWarningOverlay } from "@/components/AutoLogoutWarningOverlay";
import appCss from "../styles.css?url";

<strong>import { useOidc } from "@/oidc";
</strong>
export const Route = createRootRoute({
    head: () => ({ /* ... */ }),
    shellComponent: RootDocument
});

function RootDocument({ children }: { children: React.ReactNode }) {
    const { isOidcReady } = useOidc();

    return (
        &#x3C;html lang="en">
            &#x3C;head>
                &#x3C;HeadContent />
            &#x3C;/head>
            &#x3C;body
                className="min-h-screen text-white"
                style={{
                    backgroundColor: "#0f172a",
                    backgroundImage: "linear-gradient(180deg, #0f172a 0%, #1e293b 50%, #0f172a 100%)"
                }}
            >
                &#x3C;div className="min-h-screen flex flex-col">
<strong>                    {isOidcReady &#x26;&#x26; (
</strong><strong>                        &#x3C;>
</strong>                            &#x3C;Header />
                            &#x3C;main className="flex flex-1 flex-col">
                                &#x3C;div className="flex flex-1 flex-col">{children}&#x3C;/div>
                            &#x3C;/main>
<strong>                        &#x3C;/>
</strong><strong>                    )}
</strong>                &#x3C;/div>
                &#x3C;AutoLogoutWarningOverlay />
                &#x3C;Scripts />
            &#x3C;/body>
        &#x3C;/html>
    );
}
</code></pre>

Then, you don't need to test anymore if oidc is ready:

```diff
 function AuthButtons(props: { className?: string }) {
     const { className } = props;
-    const { isOidcReady, isUserLoggedIn } = useOidc();
+    const { isUserLoggedIn } = useOidc({ assert: "ready" });

-    if (!isOidcReady) {
-        return null;
-    }

     return (
         <div className={["opacity-0 animate-[fadeIn_0.2s_ease-in_forwards]", className].join(" ")}>
             {isUserLoggedIn ? <LoggedInAuthButton /> : <NotLoggedInAuthButton />}
         </div>
     );
 }
 
 function Greeting() {
-   const { isOidcReady, isUserLoggedIn, decodedIdToken } = useOidc();
+   const { isOidcReady, isUserLoggedIn, decodedIdToken } = useOidc({ assert: "ready" });

-   if (!isOidcReady) {
-       return <>&nbsp;</>;
-   }

    return (
        <span className="opacity-0 animate-[fadeIn_0.2s_ease-in_forwards]">
            {isUserLoggedIn ? `Welcome back ${decodedIdToken.name}` : `Hello anonymous visitor!`}
        </span>
    );
}
```
{% endtab %}
{% endtabs %}
