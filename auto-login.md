---
description: Enforce authentication everywhere in your app.
icon: shield
---

# Auto Login

Auto Login is a mode in oidc-spa designed for applications where every page requires authentication.

This is typically the case for admin dashboards or enterprise-grade apps.

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
     createOidcComponent,
     getOidc,
     oidcFnMiddleware,
     oidcRequestMiddleware,
-     enforceLogin
+    OidcInitializationGate
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
<strong>import { OidcInitializationGate } from "@/oidc";
</strong>
export const Route = createRootRoute({
    // ...
    shellComponent: RootDocument
});

function RootDocument({ children }: { children: React.ReactNode }) {
    return (
        &#x3C;html lang="en">
            &#x3C;head>
                &#x3C;HeadContent />
            &#x3C;/head>
            &#x3C;body>
                &#x3C;div className="min-h-screen flex flex-col">
                    &#x3C;Header />
                    &#x3C;main className="flex flex-1 flex-col">
<strong>                        &#x3C;OidcInitializationGate 
</strong><strong>                            pendingComponent={()=> &#x3C;Spinner />} // Optional
</strong><strong>                        >
</strong>                            {children}
<strong>                        &#x3C;/OidcInitializationGate>
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
import { createOidcComponent } from "@/oidc";

-const AuthButtons = createOidcComponent({
-    pendingComponent: () => <Spinner />,
-    component: () => {
-        const { isUserLoggedIn } = AuthButtons.useOidc();
-
-        return isUserLoggedIn ? <LoggedInAuthButton /> : <NotLoggedInAuthButton />;
-    }
-});

-const LoggedInAuthButton = createOidcComponent({
+const AuthButtons = createOidcComponent({
+    pendingComponent: () => <Spinner />,
-    assert: "user logged in",
    component: () => {
        const { logout } = LoggedInAuthButton.useOidc();

        return <button onClick={() => logout({ redirectTo: "home" })}>Logout</button>;
    }
});

-const NotLoggedInAuthButton = createOidcComponent({
-    assert: "user not logged in",
-    component: () => {
-        const { login } = NotLoggedInAuthButton.useOidc();
-
-        return <button onClick={() => login()}> Login </button>;
-    }
-});
```
{% endcode %}

You can remove the `assert: "user logged in"` from `oidcFnMiddleware` and `oidcRequestMiddleware`:

```diff
-oidcFnMiddleware({ assert: "user logged in" })
+oidcFnMiddleware({ assert: "user logged in" })

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
