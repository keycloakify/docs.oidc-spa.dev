---
description: Enforce authentication everywhere in your app.
icon: shield
---

# Auto Login

If your application requires users to be authenticated at all times—such as dashboards or admin panels—you can configure **oidc-spa** to automatically redirect unauthenticated users to the login page. This ensures that no part of your app is accessible without authentication.

This is similar to wrapping your root component with `withLoginEnforced()`, but with a key difference: **oidc-spa assumes the user will never be unauthenticated**. This means you **do not need to**:

* Check `isUserLoggedIn`, as it will always be `true`.
* (React) Use the assertion `useOidc({ assert: "user logged in" })`, since the user is guaranteed to be logged in.
* (React) Use `withLoginEnforced`, it is not exposed in this mode since it is always enforced.
* (React) You don't need to call `enforceLogin()` in your loaders.

{% tabs %}
{% tab title="Framwork Agnostic API" %}
```typescript
import { createOidc, type OidcInitializationError } from "oidc-spa";

const oidc = await createOidc({
    // ...
    autoLogin: true,
    // postLoginRedirectUrl: "/dashboard"
})
.catch((error: OidcInitializationError) => {
    // Handle potential initialization errors
    // In this mode, falling back to an unauthenticated state is not an option.

    if (!error.isAuthServerLikelyDown) {
        // This indicates a misconfiguration in your OIDC server.
        throw error;
    }

    alert("Sorry, our authentication server is down. Please try again later.");

    return new Promise<never>(() => {}); // Prevent further execution
});
```
{% endtab %}

{% tab title="React SPA" %}
{% code title="src/oidc.ts" %}
```typescript
import { createReactOidc } from "oidc-spa/react";

export const { OidcProvider, useOidc, getOidc } = createReactOidc({
   // ...
   autoLogin: true,
   // postLoginRedirectUrl: "/dashboard"
});
```
{% endcode %}

#### Handling Initialization Errors

In this mode, [initialization errors](error-management.md) must be handled at the `<OidcProvider>` level.

{% code title="src/main.tsx" %}
```tsx
import React from "react";
import ReactDOM from "react-dom/client";
import { OidcProvider } from "oidc";

const router = createRouter({ routeTree });

ReactDOM.createRoot(document.getElementById("root")!).render(
    <React.StrictMode>
        <OidcProvider
            ErrorFallback={({ initializationError }) => (
                <h1 style={{ color: "red" }}>
                    {initializationError.isAuthServerLikelyDown ? (
                        <>Sorry, our authentication server is currently down. Please try again later.</>
                    ) : (
                        // Debugging: Use initializationError.message for details.
                        // This is an issue on your end and should not be shown to users.
                        <>Unexpected authentication error.</>
                    )}
                </h1>
            )}
        >
            <RouterProvider router={router} />
        </OidcProvider>
    </React.StrictMode>
);
```
{% endcode %}
{% endtab %}

{% tab title="TanStack Start" %}
Documentation comming soon reach out [on Discord](https://discord.gg/mJdYJSdcm4) if you need this now.
{% endtab %}

{% tab title="Angular" %}
Documentation comming soon reach out [on Discord](https://discord.gg/mJdYJSdcm4) if you need this now.
{% endtab %}
{% endtabs %}

