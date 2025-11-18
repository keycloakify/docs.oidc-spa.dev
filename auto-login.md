---
description: Enforce authentication everywhere in your app.
icon: shield
---

# Auto Login

Auto Login is a mode in oidc-spa designed for applications where every page requires authentication.

This is typically the case for admin dashboards or any internal tool that do not need "marketing" pages.

When Auto Login is enabled, visiting your application automatically redirects the user to the IdP’s login page if no active session is detected.

{% tabs %}
{% tab title="Framwork Agnostic" %}
```typescript
import { createOidc } from "oidc-spa/core";

const oidc = await createOidc({
    // ...
    autoLogin: true
});
```
{% endtab %}

{% tab title="TanStack Start" %}
{% code title="src/oidc.ts" %}
```diff
 import { oidcSpa } from "oidc-spa/react-tanstack-start";
 
 export const {
     bootstrapOidc,
     useOidc,
     getOidc,
     oidcFnMiddleware,
     oidcRequestMiddleware,
-     enforceLogin
 } = oidcSpa
     .withExpectedDecodedIdTokenShape({ /* ... */ })
     .withAccessTokenValidation({ /* ... */ })
+    .withAutoLogin()
     .createUtils();
```
{% endcode %}

<pre class="language-tsx" data-title="src/routes/__root.tsx"><code class="lang-tsx">import { HeadContent, Scripts, createRootRoute } from "@tanstack/react-router";

import Header from "@/components/Header";
import { AutoLogoutWarningOverlay } from "@/components/AutoLogoutWarningOverlay";
<strong>import { useOidc } from "@/oidc";
</strong>
export const Route = createRootRoute({
    // ...
    shellComponent: RootDocument,
<strong>    ssr: false
</strong>});

function RootDocument({ children }: { children: React.ReactNode }) {

    const { isOidcReady } = useOidc();
    
    return (
        &#x3C;html lang="en">
            &#x3C;head>
                &#x3C;HeadContent />
            &#x3C;/head>
            &#x3C;body>
                &#x3C;div className="min-h-screen flex flex-col">
                    &#x3C;Header />
                    &#x3C;main className="flex flex-1 flex-col">
<strong>                        {!isOidcRead ?
</strong><strong>                            &#x3C;Spinner> :
</strong>                            children
<strong>                        }
</strong>                    &#x3C;/main>
                &#x3C;/div>
                &#x3C;AutoLogoutWarningOverlay />
                &#x3C;Scripts />
            &#x3C;/body>
        &#x3C;/html>
    );
}
</code></pre>

You can remove all the assertin your oidc component, the components specifically for the not logged in state can be removed.

{% code title="src/components/Header.tsx" %}
```diff
import { useOidc } from "@/oidc";

-function AuthButtons() {
-    const { hasInitCompleted, isUserLoggedIn } = useOidc();
-
-    if (!hasInitCompleted) {
-        return null;
-    }
-
-    return isUserLoggedIn ? <LoggedInAuthButton /> : <NotLoggedInAuthButton />;
-}
-
-function LoggedInAuthButton() {
-    const { logout } = useOidc({ assert: "user logged in" });
-
-    return (
-        <button
-            onClick={() => logout({ redirectTo: "home" })}
-        >
-            Logout
-        </button>
-    );
-}
-
-function NotLoggedInAuthButton() {
-    const { login, issuerUri } = useOidc({ assert: "user not logged in" });
-
-    return (
-        <div className="flex items-center gap-2">
-            <button
-                onClick={() => login()}
-            >
-                Login
-            </button>
-        </div>
-    );
-}

+function AuthButtons() {
+
+    const { isOidcReady, logout } = useOidc();
+
+    if (!isOidcReady) {
+        return null;
+    }
+
+    return (
+        <button
+            onClick={() => logout({ redirectTo: "home" })}
+        >
+            Logout
+        </button>
+    );
+}
```
{% endcode %}

You can remove the `assert: "user logged in"` from `oidcFnMiddleware` and `oidcRequestMiddleware`:

```diff
-oidcFnMiddleware({ assert: "user logged in" })
+oidcFnMiddleware()

-oidcRequestMiddleware({ assert: "user logged in" })
+oidcRequestMiddleware()
```

You can remove all the beforeLoad: enforceLogin:

```diff
 export const Route = createFileRoute("/demo/start/api-request")({
-    beforeLoad: enforceLogin,
     loader: async () => { }, 
     pendingComponent: () => <Spinner />,
     component: Home
 });
```

For all the components that are within the \<OidcInitializationGate /> you know that hasInitCompleted will be true so you can assert it to narrow down the type:

```diff
-const { ... } = useOidc({ assert: "user logged in" });
+const { ... } = useOidc({ assert: "ready" });
```
{% endtab %}

{% tab title="React SPA" %}
{% code title="src/oidc.ts" %}
```diff
 export const {
     bootstrapOidc,
     useOidc,
     getOidc,
     OidcInitializationGate
-    withLoginEnforced,
-    enforceLogin
 } = oidcSpa
     .withExpectedDecodedIdTokenShape({ /* ... */ })
+    .withAutoLogin()
     .createUtils();
```
{% endcode %}

You can then proceed to remove all the usage of `withLoginEnforced` and `enforceLogin` throughout your codebase.\
\
You can also remove all the assetion of the login state of the user:

```diff
- useOidc({ assert: "user logged in" });
+ useOidc();
```

All the components with `useOidc({ assert: "user not logged in" });` can be removed.
{% endtab %}

{% tab title="Angular" %}
<pre class="language-typescript" data-title="src/app/services/oidc.service.ts"><code class="lang-typescript">@Injectable({ providedIn: 'root' })
export class Oidc extends AbstractOidcService&#x3C;DecodedIdToken> {
  // ...
<strong>  override autoLogin = true;
</strong>}
</code></pre>

All the handling for the user not logged in state can be removed.

{% code title="src/app/app.html" %}
```diff
-@if (oidc.isUserLoggedIn) {
 <div>
       <span>Hello {{ oidc.$decodedIdToken().name }}</span>
       <button (click)="oidc.logout({ redirectTo: 'home' })">Logout</button>
 </div>
-} @else {
-<div>
-      <button (click)="oidc.login()">Login</button>
-</div>
-}
```
{% endcode %}

You can remove the usage of Oidc.enforceLoginGuard:

{% code title="src/app/app.routes.ts" %}
```diff
-canActivate: [Oidc.enforceLoginGuard],
//...
 canActivate: [
   async (route) => {
     const oidc = inject(Oidc);
     const router = inject(Router);
-    await Oidc.enforceLoginGuard(route);   
     //...
  },
],
```
{% endcode %}
{% endtab %}
{% endtabs %}
