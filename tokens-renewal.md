---
icon: arrows-rotate-reverse
---

# Tokens Renewal

`oidc-spa` completely abstracts away the concern of refreshing tokens; you don’t need to handle it yourself.

However, it still provides a `renewTokens()` function for a couple of rare but valid edge cases:

* **Force update after custom requests**: If you send a custom request to your OIDC server (e.g., Keycloak) that you know has changed some claims in the `id_token` or `access_token`, you can call `renewTokens()` to ensure you’re working with the latest values.
* **Custom parameters to the token endpoint**: If your OIDC server’s token endpoint accepts special parameters, `renewTokens()` lets you trigger a refresh that including custom parameters (there is also the `extraTokenParams` option that you can provide to `createOidc`.).

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
