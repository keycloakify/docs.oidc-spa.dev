# 🛡️ Enforce Authentication Globally

If there is no part of your app that can be accessed without being logged it you can make oidc-spa automatically redirect your users to the login pages when they are not authenticated. &#x20;

Note that in this mode you don't have to check `isUserLoggedIn` or `useOidc({ assertUserLoggedIn: true })`.

{% tabs %}
{% tab title="Vanilla API" %}
```typescript
import { createOidc, OidcInitializationError } from "oidc-spa";

try{

const oidc = await createOidc({
    // ...
   isAuthGloballyRequired: true,
   // Optional, the default value is: location.href (here)
   // postLoginRedirectUrl: "/dashboard"
});

}catch(error){
    const oidcInitializationError = error as OidcInitializationError;
    console.log(oidcInitializationError.message);
    console.log(oidcInitializationError.type); // "server down" | "bad configuration" | "unknown";
}
```
{% endtab %}

{% tab title="React API" %}
{% code title="src/oidc.ts" %}
```typescript
import { createReactOidc } from "oidc-spa/react";

export const {
    OidcProvider,
    useOidc,
    prOidc
} = createReactOidc({
   // ...
   isAuthGloballyRequired: true,
   // Optional, the default value is: location.href (here)
   // postLoginRedirectUrl: "/dashboard"
});
```
{% endcode %}

{% code title="src/main.tsx" %}
```tsx

import { OidcProvider } from "oidc";

ReactDOM.createRoot(document.getElementById("root")!).render(
    <React.StrictMode>
        <OidcProvider 
            ErrorFallback={({ initializationError })=>(
                <h1 style={{ color: "red" }}>
                    An error occurred while initializing the OIDC client:
                    {initializationError.message}
                    {initializationError.type} /* "server down" | "bad configuration" | "unknown"; */
                </h1>
            )}
        >
            {/* ... */}
        </OidcProvider>
    </React.StrictMode>
);

```
{% endcode %}
{% endtab %}
{% endtabs %}
