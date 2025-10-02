---
icon: arrows-rotate-reverse
---

# Tokens Renewal

Nearly all OpenID Connect adapters, including `keycloak-js`, implement token renewal incorrectly.\
With `oidc-spa`, you never have to worry about this. Token lifecycle management is fully abstracted away, as it should be.

***

**The Problem With Access Token Refresh Loops**

Access tokens are meant to be **short-lived** (typically \~5 minutes, but sometimes as little as 20 seconds for high-security apps).\
Many adapters try to **keep an access token “always fresh” in cache**, which leads to:

* Constant background refreshes
* Heavy load on your auth server
* Wasteful duplication when multiple tabs are open

This is unnecessary. You don’t need a valid access token cached at all times.

***

**The Correct Approach (What `oidc-spa` Does)**

Whenever you need to make an authenticated request, just **ask `oidc-spa` for a token**:

```ts
const oidc = await getOidc();

if (!oidc.isUserLoggedIn) {
    throw Error("Logical error in our application flow");
}

const { accessToken } = await oidc.getTokens();
headers.set("Authorization", `Bearer ${accessToken}`);
```

* If a valid token is cached, you’ll get it.
* If it’s expired, `oidc-spa` silently refreshes it using the refresh token.\


So the correct approaches when you need an access token are:

* via an interceptor that injects it into requests, or
* with a wrapper around `fetch()` that awaits `oidc.getAccessToken()`.

Example: [interceptor pattern](https://github.com/InseeFrLab/onyxia/blob/2f7bad234099719debc15ecdaba30dba116ffef9/web/src/core/adapters/onyxiaApi/onyxiaApi.ts#L34-L84)\
Example: [custom fetch](https://github.com/keycloakify/oidc-spa/blob/a1aae19e2b5a874159fbdfecaaf00be814bb4c6a/examples/tanstack-router-file-based/src/oidc.tsx#L64-L76)

Behind the scenes, `oidc-spa` ensures the session **never expires prematurely** by refreshing **at least once before the refresh token itself expires**.\
This prevents the backend from destroying the session simply because the user wasn’t making authenticated requests (e.g., they’re filling out a form or browsing content).

At the same time, `oidc-spa` tracks **actual user activity** (keyboard, mouse, touch). If the user is truly idle beyond the refresh token lifespan, they’re logged out as expected.

This is the **only correct model**. There aren’t “multiple valid strategies.”\
It shouldn’t be configurable, because there is nothing to configure.

***

**Why Other Adapters Get It Wrong**

Adapters like `keycloak-js` expose the access token synchronously (`keycloak.token`).\
To keep that contract, they’re forced to **brute-force the server with background refreshes,** otherwise, you might read an expired token.

That’s why they give you knobs to “configure auto-renewal.” In reality, this pushes responsibility onto you for something the adapter should handle internally.

On top of that the fresh loop does not even guarenty you'll never read an expired token, for example, when the computer wakes up from sleep to an expired token, the token can be expired and the adapter would have had no time to refresh it.  \
\
The Keycloak team is well aware of the design flaw of keycloak-js and keep it as is because changing the API would cause too much distruption. However, when they use keycloak-js internally for their app, like the Admin Console. [They apply the same strategy as oidc-spa](https://github.com/keycloak/keycloak/blob/f53e5ebdac5ca40bc5a232bb71b3cdd59670e1ee/js/apps/admin-ui/src/admin-client.ts#L29-L38). &#x20;

***

**Why `oidc-spa` Still Exposes `renewTokens()`**

There are two legitimate edge cases:

1. **After custom requests**: If you make a request to your OIDC server that changes claims in the `id_token` or `access_token`, call `renewTokens()` to ensure you have the latest values. (This is a very rare usecase, usually the user info are updated outside of your app, if you're not sure, you can safely assume your not doing any of those request). &#x20;
2. **Custom token parameters**: If your OIDC server supports extra token endpoint params, you can trigger a refresh with them. (`extraTokenParams` is also available at `createOidc()` time.)

Outside of these rare cases, you never need to call `renewTokens()` manually.

***

👉 With `oidc-spa`, token renewal is **always correct, efficient, and invisible to you**.

{% tabs %}
{% tab title="Vanilla API" %}
```typescript
import { createOidc } from "oidc-spa";

const prOidc = await createOidc({ ... });

// Function to call when we want to renew the token
export function renewTokens(){

   const oidc = await prOidc;
   
   if( !oidc.isUserLoggedIn ){
      throw new Error("Logical error");
   }
   
   oidc.renewToken(
      // Optionally you can pass extra params that will be added 
      // to the body of the POST request to the openid-connect/token endpoint.
      // { extraTokenParams: { electedCustomer: "customer123" } }
      // This parameter can also be provided as parameter to the createOidc
      // function. See: https://github.com/keycloakify/oidc-spa/blob/59b8db7db0b47c84e8f383a86677e88e884887cb/src/oidc.ts#L153-L163
   );

}

// Subscribing to token renewal

prOidc.then(oidc => {
    if( !oidc.isUserLoggedIn ){
        return;
    }
    
    const { unsubscribe } = oidc.subscribeToTokensChange(tokens => {
       console.log("Token Renewed", tokens);
    });
    
    setTimeout(() => {
        // Call unsubscribe when you want to stop watching tokens change
        unsubscribe();
    }, 10_000);
});
```
{% endtab %}

{% tab title="React API" %}
Outside of a React Component:&#x20;

```typescript
import { createReactOidc } from "oidc-spa/react";

const prOidc = await createReactOidc({ ... });

// Function to call when we want to renew the token
export function renewTokens(){

   const oidc = await prOidc;
   
   if( !oidc.isUserLoggedIn ){
      throw new Error("Logical error");
   }
   
   oidc.renewToken(
      // Optionally you can pass extra params that will be added 
      // to the body of the POST request to the openid-connect/token endpoint.
      // { extraTokenParams: { electedCustomer: "customer123" } }
      // This parameter can also be provided as parameter to the createOidc
      // function. See: https://github.com/keycloakify/oidc-spa/blob/59b8db7db0b47c84e8f383a86677e88e884887cb/src/oidc.ts#L153-L163
   );

}

// Subscribing to token renewal

prOidc.then(oidc => {
    if( !oidc.isUserLoggedIn ){
        return;
    }
    
    const { unsubscribe } = oidc.subscribeToTokensChange(tokens => {
       console.log("Token Renewed", tokens);
    });
    
    setTimeout(() => {
        // Call unsubscribe when you want to stop watching tokens change
        unsubscribe();
    }, 10_000);
});
```

Inside of a React Component (not valid for real world usecase, just for diagnostic)

```tsx
import { useState, useEffect } from "react";
import { assert } from "tsafe/assert";
import { useOidc, getOidc } from "../oidc";
import { decodeJwt } from "oidc-spa/tools/decodeJwt";

export function LogTokens() {
    const { decodedIdToken, renewTokens } = useOidc({ assert: "user logged in" });

    const { decodedAccessToken } = useDecodedAccessToken();

    if (decodedAccessToken === undefined) {
        // Loading...
        return null;
    }

    return (
        <>
            <h3>Decoded ID Token:</h3>
            <pre>{JSON.stringify(decodedIdToken, null, 2)}</pre>
            <h3>Decoded Access Token:</h3>
            <pre>{JSON.stringify(decodedAccessToken, null, 2)}</pre>
            <br />
            <button onClick={() => renewTokens()}>Refresh Tokens</button>
        </>
    );
}

function useDecodedAccessToken() {
    const [decodedAccessToken, setDecodedAccessToken] = useState<
        Record<string, unknown> | undefined /* Loading */
    >(undefined);

    useEffect(() => {
        let cleanup: (() => void) | undefined = undefined;
        let isActive = true;

        (async () => {
            const oidc = await getOidc();

            if (!isActive) {
                return;
            }

            assert(oidc.isUserLoggedIn);

            const update = (accessToken: string) => setDecodedAccessToken(decodeJwt(accessToken));

            const { unsubscribe } = oidc.subscribeToTokensChange(tokens => update(tokens.accessToken));

            cleanup = () => {
                unsubscribe();
            };

            {
                const { accessToken } = await oidc.getTokens();

                if (!isActive) {
                    return;
                }

                update(accessToken);
            }
        })();

        return () => {
            isActive = false;
            cleanup?.();
        };
    }, []);

    return { decodedAccessToken };
}

```
{% endtab %}
{% endtabs %}
