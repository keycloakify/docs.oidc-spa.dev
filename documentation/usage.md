---
description: Let's get your App authenticated!
---

# 👨‍🔧 Basic Usage

{% hint style="info" %}
In this guide, we assume that you have an OIDC-enabled authentication server in place, such as Keycloak.&#x20;

If you have not yet set up such a server, please refer to [our guide for instructions on how to provision and configure a Keycloak server](../resources/usage-with-keycloak.md).
{% endhint %}

{% tabs %}
{% tab title="Vanilla API" %}
```typescript
import { createOidc } from "oidc-spa";

const oidc = await createOidc({
    issuerUri: "https://auth.your-domain.net/realms/myrealm",
    clientId: "myclient",
    /**
     * Vite:  `publicUrl: import.meta.env.BASE_URL`
     * CRA:   `publicUrl: process.env.PUBLIC_URL`
     * Other: `publicUrl: "/"` (Usually)
     */
    publicUrl: import.meta.env.BASE_URL
});

if (!oidc.isUserLoggedIn) {
    // The user is not logged in.

    // We can call login() to redirect the user to the login/register page.
    // This return a promise that never resolve. 
    oidc.login({
         /** 
          * If you are calling login() in the callback of a click event
          * set this to false.  
          */
         doesCurrentHrefRequiresAuth: false
         /** 
          * Optionally, you can add some extra parameter 
          * to be added on the login url.  
          * (This can also be a parameter of createOidc)
          */
         //extraQueryParams: { kc_idp_hint: "google", kc_locale: "fr" }
         /**
          * You can allso set where to redirect the user after 
          * successful login
          */
          // redirectUrl: "/dashboard"
    });

} else {
    // The user is logged in.

    const {
        // The accessToken is what you'll use as a Bearer token to 
        // authenticate to your APIs
        accessToken,
        decodedIdToken
    } = oidc.getTokens();

    fetch("https://api.your-domain.net/orders", {
        headers: {
            Authorization: `Bearer ${accessToken}`
        }
    })
     .then(response => response.json())
     .then(orders => console.log(orders));

    // To call when the user click on logout.
    // You can also redirect to a custom url with 
    // { redirectTo: "specific url", url: "/bye" }
    oidc.logout({ redirectTo: "home" });

    // If you are wondering why ther's a decodedIdToken and no
    // decodedAccessToken read this: https://docs.oidc-spa.dev/resources/jwt-of-the-access-token
    console.log(`Hello ${decodedIdToken.preferred_username}`);

    // Note that in this example the decodedIdToken is not typed.  
    // What is inside the idToken is defined by the OIDC server you are using.  
    // If you want to specify the type of the decodedIdToken you can do:
    //
    // type DecodedIdToken = {
    //    sub: string;
    //    preferred_username: string;
    //    // ...
    // };
    //
    // export const { useOidc } = createUseOidc<DecodedIdToken>(...)
    //
    // If you want the shape of the decodedIdToken to be validated at runtime
    // you can provide a validator function using zod for example:
    //
    // export const { useOidc } = createUseOidc<DecodedIdToken>({
    //    ...
    //    decodedIdTokenSchema: z.object({
    //        sub: z.string(),
    //        preferred_username: z.string()
    //    })
    // })

}
```
{% endtab %}

{% tab title="React API" %}
This piece of code should give you the necessary information to understand how oidc-spa can be used inside your react components.  \
To go further you can refer to the examples setup to see how to integrate oidc-spa with your routing library: &#x20;

* [react-router-dom example setup](../example-setups/react-router.md)
* [@tanstack/react-router example setup](../example-setups/tanstack-router.md)

```tsx
import { createReactOidc } from "oidc-spa/react";

export const { OidcProvider, useOidc } = createReactOidc({
    // NOTE: If you don't have the params right away see note below.
    issuerUri: "https://auth.your-domain.net/realms/myrealm",
    clientId: "myclient",
    /**
     * Vite:  `publicUrl: import.meta.env.BASE_URL`
     * CRA:   `publicUrl: process.env.PUBLIC_URL`
     * Other: `publicUrl: "/"` (Usually)
     */
    publicUrl: import.meta.env.BASE_URL
});

ReactDOM.createRoot(document.getElementById("root")!).render(
    <OidcProvider
        // Optional, it's usually so fast that a fallback is really not required.
        fallback={<>Checking authentication ⌛️</>}
    >
        <App />
    </OidcProvider>
);

function App() {

    const { isUserLoggedIn, login, logout, oidcTokens } = useOidc();

    return (
        isUserLoggedIn ? (
            <>
                {/* 
                Note: The decodedIdToken can be typed and validated with zod See: https://github.com/keycloakify/oidc-spa/blob/fddac99d2b49669a376f9a0b998a8954174d195e/examples/tanstack-router/src/oidc.tsx#L17-L43
                If you are wondering why ther's a decodedIdToken and no
                decodedAccessToken read this: https://docs.oidc-spa.dev/resources/jwt-of-the-access-token
                */}
                <span>Hello {oidcTokens.decodedIdToken.preferred_username}</span>
                <button onClick={() => logout({ redirectTo: "home" })}>
                  Logout
                </button>
            </>
        ) : (
            <button onClick={() => login({ 
                /** 
                 * If you are calling login() in the callback of a button click
                 * (like here) set this to false.  
                 */
                doesCurrentHrefRequiresAuth: false
                /** 
                 * Optionally, you can add some extra parameter 
                 * to be added on the login url.
                 * (Can also be a parameter of createReactOidc)
                 */
                //extraQueryParams: { kc_idp_hint: "google", kc_locale: "fr" }
                /**
                 * You can allso set where to redirect the user after 
                 * successful login
                 */
                // redirectUrl: "/dashboard"
            })} >
              Login
            </button>
        )
    );
}

import { useEffect, useState } from "react";

type Order = {
  id: number;
  name: string;
};

function OrderHistory(){

    const { oidcTokens } = useOidc({ assertUserLoggedIn: true });

    const [orders, setOrders] = useState<Order[] | undefined>(undefined);

    useEffect(
        ()=> {

            fetch("https://api.your-domain.net/orders", {
                headers: {
                    Authorization: `Bearer ${oidcTokens.accessToken}`
                }
            })
            .then(response => response.json())
            .then(orders => setOrders(orders));

        },
        []
    );

    if(orders === undefined){
        return <>Loading orders ⌛️</>
    }

    return (
        <ul>
            {orders.map(order => (
                <li key={order.id}>{order.name}</li>
            ))}
        </ul>
    );

}
```

If you get your OIDC parameters from an API you can passes a promise that resolves to the params to `createReactOidc`:

```typescript
const { /* ... */ } = createReactOidc((async ()=> {

    const { 
        issuerUri, 
        clientId 
    } = await axios.get("/oidc-params").then(r => r.data);

    return {
        issuerUri,
        clientId,
        publicUrl: import.meta.env.BASE_URL
    };
    
})());
```
{% endtab %}
{% endtabs %}
