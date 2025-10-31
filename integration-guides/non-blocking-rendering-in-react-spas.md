---
icon: gauge-max
---

# Non Blocking Rendering in React SPAs

If you are using the idc-spa/react-spa adapter, the official instruction is to wrap your all application within \<OidcInitializationGate /> like:&#x20;

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

However what this does is that it will defer the rendering of your whole app until `boostrapOidc()` has resolved. That is to say, after oidc-spa has contacted the IdP and established wether the user has an active session or not. &#x20;

This might very well be what you want. This simplify your mentale model, you don't have to think about wether or not the auth state have been settled yet. You also avoid any potential layout shift.  \
\
Now, for optimal performances you might want to start rendering even before the auth state is settled. To acheive result like this:

{% embed url="https://youtu.be/t1qfU_GeTM4?si=xrbRvl9dJQS9xccJ" %}

In this short video we can see that the homepage renders instantly then the auth aware component apears subsequently when auth state is settled.  \
\
You can achive this simply by moving the \<OidcInitializationGate /> closer to the components that call useOidc(). In practice it will look a bit like this:

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
</strong><strong>            {/* OR */}
</strong><strong>            {/*
</strong><strong>            &#x3C;Suspense fallback={&#x3C;Spinner />}>
</strong>                &#x3C;AuthButtons />
            &#x3C;/Suspense>
<strong>            */}
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

You can also use React's built in \<Suspense /> instead of \<OidcInitializationGate />, it's even beter since it will let you have an unified fallback for all the loading opterations. useOidc(), when called while the auth state is not yet settled will throw a promise, that will be caugh by the nearest suspense boundary.

This also mean that you must make sure to wrap every components that uses useOidc into \<OidcInitializationGate /> or \<Suspense /> or your all app will suspend (Don't forget the \<AutoLogoutWarningOverlay />).  \
\
Note however that any component protected with enforceLogin or within a component wrapped in withLoginEnforced() will never suspend since those act as authentication gate themselvs.  \
\
Last thing to take into consideration is if you use withLoginEnforced() note that component using this HOC can suspend as well. So if you use them you want to wrap them into \<OidcInitializationGate /> or \<Suspense /> as well. Example:

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

With all the routes components like:

{% code title="src/pages/Protected.tsx" %}
```tsx
import { withLoginEnforced } from "~/oidc";

const Protected = withLoginEnforced(() => {
    return <div>{/* ... */}</div>;
});

export default Protected;
```
{% endcode %}
