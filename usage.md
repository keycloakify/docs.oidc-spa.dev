---
description: Let's get your App authenticated!
icon: flag-checkered
---

# Basic Usage

Before getting started, you need to get a hold of the few parameters required to connect to your OIDC provider.\
Find instruction on how to configure your OIDC provider on the following documentation page:

{% content-ref url="providers-configuration/provider-configuration.md" %}
[provider-configuration.md](providers-configuration/provider-configuration.md)
{% endcontent-ref %}

{% tabs %}
{% tab title="Vanilla API" %}
```typescript
import { createOidc } from "oidc-spa";
import { z } from "zod";

const oidc = await createOidc({
    issuerUri: "https://auth.your-domain.net/realms/myrealm",
    clientId: "myclient",
    /**
     * Vite:  `homeUrl: import.meta.env.BASE_URL`
     * CRA:   `homeUrl: process.env.PUBLIC_URL`
     * Other: `homeUrl: "/"` (Usually, can be something like "/dashboard")
     */
    homeUrl: import.meta.env.BASE_URL,
    //scopes: ["profile", "email", "api://my-app/access_as_user"],
    extraQueryParams: () => ({
       ui_locales: "en" // Keycloak login/register page language
       //audience: "https://my-app.my-company.com/api"
     }),
     decodedIdTokenSchema: z.object({
        preferred_username: z.string(),
        name: z.string()
        //email: z.string().email().optional()
     })
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
          * (Can also be a parameter of createOidc `extraQueryParams: ()=> ({ ui_locales: "fr" })`)
          */
         //extraQueryParams: { kc_idp_hint: "google", ui_locales: "fr" }
         /**
          * You can allso set where to redirect the user after 
          * successful login
          */
          // redirectUrl: "/dashboard"
          
          /**
           * Keycloak: You can also send the users directly to the register page
           * see: https://github.com/keycloakify/oidc-spa/blob/14a3777601c50fa69d1221495d77668e97443119/examples/tanstack-router-file-based/src/components/Header.tsx#L54-L66
           */ 
    });

} else {
    // The user is logged in.

    const {
        // The accessToken is what you'll use as a Bearer token to 
        // authenticate to your APIs
        accessToken
    } = await oidc.getTokens();
    
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
    
    const decodedIdToken = oidc.getDecodedIdToken();

    console.log(`Hello ${decodedIdToken.preferred_username}`);

}
```
{% endtab %}

{% tab title="React API" %}
The way you use **oidc-spa** differs slightly depending on the routing library you’re using (e.g., React Router or TanStack Router).\
We provide specific [setup guides](setup-guides/example-setups.md) for each, but we recommend starting with the fictional example below to understand how the library works in isolation, without any routing-related distractions.

Note: In this example, some pages can be accessed without requiring the user to be authenticated.\
If you're building something like an admin panel or a dashboard where authentication is always required, simply set [`autoLogin: true`](auto-login.md).

```
src/
├── components/
│   └── Header.tsx
├── pages/
│   ├── Home.tsx
│   ├── Account.tsx
│   ├── Orders.tsx
├── App.tsx
└── oidc.tsx
```

{% code title="src/oidc.ts" %}
```typescript
import { createReactOidc } from "oidc-spa/react";
import { z } from "zod";

export const { OidcProvider, useOidc, getOidc, withLoginEnforced, enforceLogin } =
    createReactOidc(async () => ({
        issuerUri: "https://auth.your-domain.net/realms/myrealm",
        clientId: "myclient",
        /**
         * Vite:  `homeUrl: import.meta.env.BASE_URL`
         * CRA:   `homeUrl: process.env.PUBLIC_URL`
         * Other: `homeUrl: "/"` (Usually, can be something like "/dashboard")
         */
        homeUrl: import.meta.env.BASE_URL,
        //scopes: ["profile", "email", "api://my-app/access_as_user"],
        extraQueryParams: () => ({
            ui_locales: "en" // Keycloak login/register page language
            //audience: "https://my-app.my-company.com/api"
        }),
        decodedIdTokenSchema: z.object({
            preferred_username: z.string(),
            name: z.string()
            //email: z.string().email().optional()
        })
    }));

export const fetchWithAuth: typeof fetch = async (
    input,
    init
) => {
    const oidc = await getOidc();
    
    if (oidc.isUserLoggedIn) {
        const { accessToken } = await oidc.getTokens();

        (init ??= {}).headers = {
            ...init.headers,
            Authorization: `Bearer ${accessToken}`
        };
    }

    return fetch(input, init);
};
```
{% endcode %}

{% code title="src/App.tsx" %}
```tsx
import { Suspense, lazy } from "react";
import Header from "./components/Header";
const HomePage = lazy(() => import("./pages/Home"));
const OrderPage = lazy(() => import("./pages/Orders"));
const AccountPage = lazy(() => import("./pages/Account"));

export default function App() {
    const route = useRoute();

    return (
        <OidcProvider
          //fallback={<h1>Checking authentication ⌛️</h1>}
        >
            <Header />
            <main>
                <Suspense>
                    {route === "/home" && <HomePage />}
                    {route === "/orders" && <OrderPage />}
                    {route === "/account" && <AccountPage />}
                </Suspense>
            </main>
        </OidcProvider>
    );
}
```
{% endcode %}

{% code title="src/components/Header.tsx" %}
```tsx
import { useOidc } from "../oidc";

export default function Header() {
    const { isUserLoggedIn } = useOidc();

    return (
        <header>
            <nav>
                <Link to="/home">Home</Link>
                <Link to="/orders">Orders</Link>
            </nav>
            {isUserLoggedIn ? (
                <AuthButtonsLoggedIn />
            ) : (
                <AuthButtonsNotLoggedIn />
            )}
        </header>
    );
}

function AuthButtonsLoggedIn() {
    const { decodedIdToken, logout } = useOidc({ assert: "user logged in" });

    return (
        <div>
            <span>Logged in as {decodedIdToken.preferred_username}</span>
            <Link to="/account">Account</Link>
            <button onClick={() => logout({ redirectTo: "home" })}>
                Logout
            </button>
        </div>
    );
}

function AuthButtonsNotLoggedIn() {
    const { login } = useOidc({ assert: "user not logged in" });

    return (
        <div>
            <button onClick={() => login()}>Login</button>
            <button
                onClick={() =>
                    login({
                        // Keycloak:
                        transformUrlBeforeRedirect: url => {
                            const urlObj = new URL(url);
                            urlObj.pathname = urlObj.pathname.replace(
                                /\/auth$/,
                                "/registrations"
                            );
                            return urlObj.href;
                        }
                        // Auth0:
                        // extraQueryParams: { screen_hint: "signup" }
                    })
                }
            >
                Register
            </button>
        </div>
    );
}
```
{% endcode %}

{% code title="src/pages/Home.tsx" %}
```tsx
import { useOidc } from "../oidc";

export default function Page() {
    const { isUserLoggedIn, decodedIdToken } = useOidc();

    return (
        <h1>Welcome {isUserLoggedIn ? decodedIdToken.name : "guest"}!</h1>
    );
}
```
{% endcode %}

{% code title="src/pages/Orders.tsx" %}
```tsx
import { useEffect, useState } from "react";
import { withLoginEnforced, fetchWithAuth } from "../oidc";

type Order = {
    id: number;
    name: string;
};

// If this component is mounted and the user is not logged in
// the user will be redirected to the login.  
// If your routing library support loader you can use enforceLogin
// instead of withLoginEnforced
const Page = withLoginEnforced(() => {
    const [orders, setOrders] = useState<Order[] | undefined>(undefined);

    useEffect(() => {
        fetchWithAuth("https://api.your-domain.net/orders", {
            headers: {
                "Content-Type": "application/json"
            }
        })
            .then(response => response.json())
            .then(orders => setOrders(orders));
    }, []);

    if (orders === undefined) {
        return <>Loading orders ⌛️</>;
    }

    return (
        <ul>
            {orders.map(order => (
                <li key={order.id}>{order.name}</li>
            ))}
        </ul>
    );
});

export default Page;
```
{% endcode %}

{% code title="src/pages/Account.tsx" %}
```tsx
import { useOidc, withLoginEnforced } from "../oidc";
import { parseKeycloakIssuerUri } from "oidc-spa/tools/parseKeycloakIssuerUri";

const Page = withLoginEnforced(() => {
    const {
        goToAuthServer,
        backFromAuthServer,
        params: { issuerUri, clientId }
    } = useOidc({ assert: "user logged in" });

    const keycloak = parseKeycloakIssuerUri(issuerUri);

    if (keycloak === undefined) {
        throw new Error(
            "We expect Keycloak to be the OIDC provider of this App"
        );
    }

    return (
        <div>
            <h1>Account</h1>
            <p>
                <a
                    href={keycloak.getAccountUrl({
                        clientId,
                        backToAppFromAccountUrl: location.href,
                        locale: "en"
                    })}
                >
                    Go to Keycloak Account Management Page
                </a>
            </p>
            <p>
                <button
                    onClick={() =>
                        goToAuthServer({
                            extraQueryParams: {
                                kc_action: "UPDATE_PASSWORD"
                            }
                        })
                    }
                >
                    Change My Password
                </button>
                {backFromAuthServer?.extraQueryParams.kc_action ===
                    "UPDATE_PASSWORD" && (
                    <span>
                        {backFromAuthServer.result.kc_action_status ===
                        "success"
                            ? "Password Updated!"
                            : "Password unchanged"}
                    </span>
                )}
            </p>
            <p>
                <button
                    onClick={() =>
                        goToAuthServer({
                            extraQueryParams: {
                                kc_action: "UPDATE_PROFILE"
                            }
                        })
                    }
                >
                    Update My Profile Information
                </button>
                {backFromAuthServer?.extraQueryParams.kc_action ===
                    "UPDATE_PROFILE" && (
                    <span>
                        {backFromAuthServer.result.kc_action_status ===
                        "success"
                            ? "Profile Updated!"
                            : "Profile unchanged"}
                    </span>
                )}
            </p>
            <p>
                <button
                    onClick={() =>
                        goToAuthServer({
                            extraQueryParams: {
                                kc_action: "delete_account"
                            }
                        })
                    }
                >
                    Delete My Account
                </button>
            </p>
        </div>
    );
});

export default Page;
```
{% endcode %}

Now that you got the idea you can follow up with the specific setup guides for different stacks:

{% content-ref url="setup-guides/example-setups.md" %}
[example-setups.md](setup-guides/example-setups.md)
{% endcontent-ref %}
{% endtab %}
{% endtabs %}
