---
icon: gauge-max
---

# Non Blocking Rendering in React SPAs

When using the `oidc-spa/react-spa` adapter, the recommended setup is to wrap your entire application in an `<OidcInitializationGate />`, like so:

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
* Tests and SSR behave predictably (SSR is canceled).

***

However, for **optimal performance**, you can start rendering _before_ the authentication state is resolved, letting the page appear instantly, while auth-aware components hydrate a few milliseconds later.

For example:

{% embed url="https://youtu.be/t1qfU_GeTM4?si=xrbRvl9dJQS9xccJ" %}

In this short demo, the homepage renders immediately, and components depending on authentication appear shortly after the session check completes.

You can achieve this simply by moving `<OidcInitializationGate />` closer to the components that call `useOidc()`:

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
</strong>
<strong>            {/* OR */}
</strong>
<strong>            {/*
</strong>            &#x3C;Suspense fallback={&#x3C;Spinner />}>
                &#x3C;AuthButtons />
            &#x3C;/Suspense>
<strong>            */}
</strong>
        &#x3C;/header>
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

You can use React’s built-in `<Suspense />` instead of `<OidcInitializationGate />`.\
This is often even better, as it lets you define a unified fallback for all your app’s asynchronous operations.

When called before the auth state is ready, `useOidc()` throws a Promise, which React will catch using the nearest Suspense boundary.

This means you **must** wrap any component that calls `useOidc()` in either `<OidcInitializationGate />` or `<Suspense />`.\
If you don’t, your entire app will suspend.\
&#xNAN;_(And don’t forget to wrap `<AutoLogoutWarningOverlay />` as well.)_

***

### Components protected with `enforceLogin`

Any component that’s behind `enforceLogin()` or wrapped in a  `withLoginEnforced()` component **will not suspend**, because those act as their own authentication gates.\
However, note that if you use `withLoginEnforced()` directly, the resulting component can still suspend, so you want to wrap them too.

Example:

<pre class="language-tsx" data-title="src/App.tsx"><code class="lang-tsx">import { lazy, Suspense } from "react";
import { Navigate, Route, Routes } from "react-router";
import { AutoLogoutWarningOverlay } from "./components/AutoLogoutWarningOverlay";
import { Header } from "./components/Header";
import { Home } from "./pages/Home";
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
            &#x3C;Suspense>
                &#x3C;AutoLogoutWarningOverlay />
            &#x3C;/Suspense>
        &#x3C;/>
    );
}
</code></pre>

With route components like:

{% code title="src/pages/Protected.tsx" %}
```tsx
import { withLoginEnforced } from "~/oidc";

const Protected = withLoginEnforced(() => {
    return <div>{/* ... */}</div>;
});

export default Protected;
```
{% endcode %}

***

### TL;DR

* `<OidcInitializationGate />` at the root: **simpler mental model**, no layout shift.
* `<Suspense />` or `<OidcInitializationGate />` near `useOidc()` calls: **faster perceived load**, better user experience.
* `enforceLogin` and `withLoginEnforced()` automatically handle suspension but Page component wrapped into `withLoginEnforced()` do suspend themselvs.
* Both options are supported choose based on your desired UX and simplicity.

***

_(In modern browsers, session restoration typically takes under 300 ms, so even full gating often feels instant.)_
