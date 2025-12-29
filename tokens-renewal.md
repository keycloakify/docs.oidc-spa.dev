---
icon: arrows-rotate-reverse
---

# Tokens Renewal

Many OpenID Connect adapters, end up implementing token renewal with a background refresh loop.\
That approach often creates avoidable load and some tricky edge cases.\
With `oidc-spa`, token lifecycle management is handled for you and stays out of your app code.

***

**The Problem With Access Token Refresh Loops**

Access tokens are meant to be **short-lived** (typically \~5 minutes, but sometimes as little as 20 seconds for high-security apps).\
Many adapters try to **keep an access token “always fresh” in cache**, which leads to:

* Constant background refreshes
* Heavy load on your auth server
* Agravated load when mutiple tabs are open on your app.

This isn’t needed. You don’t need a valid access token cached at all times.

***

**The Better Approach (What `oidc-spa` Does)**

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
* If it’s expired or soon to expire, `oidc-spa` silently refreshes it using the refresh token.

Example: [interceptor pattern](https://github.com/InseeFrLab/onyxia/blob/2f7bad234099719debc15ecdaba30dba116ffef9/web/src/core/adapters/onyxiaApi/onyxiaApi.ts#L34-L84)\
Example: [custom fetch](https://github.com/keycloakify/oidc-spa/blob/a1aae19e2b5a874159fbdfecaaf00be814bb4c6a/examples/tanstack-router-file-based/src/oidc.tsx#L64-L76)

**But what about session expiration?**&#x20;

Behind the scenes, `oidc-spa` ensures the session **never expires prematurely** by refreshing **at least once before the refresh token itself expires**.  \
This prevents the backend from destroying the session simply because the user wasn’t making authenticated requests (e.g., they’re filling out a form or browsing content). &#x20;

At the same time, `oidc-spa` tracks **actual user activity** (keyboard, mouse, touch). If the user is truly idle beyond the refresh token lifespan, they’re logged out as expected.

***

**Why `oidc-spa` Still Exposes `renewTokens()`**

There are two legitimate edge cases:

1. **After custom requests**: If you make a request to your OIDC server that changes claims in the `id_token` or `access_token`, call `renewTokens()` to ensure you have the latest values. (This is a rare use case. It usually happens when user info is updated outside your app. If you’re not sure, you can generally assume you don’t need this.)
2. Getting a freshly issued token: If at one point in time, you want to be sure that you have a freshly issued token with it's maximum lifetime you might want to call renewTokens() before you call getTokens()
3. **Custom token parameters**: If your OIDC server supports extra token endpoint params, you can trigger a refresh with them. (`extraTokenParams` is also available at `createOidc()` time.)

Outside of these rare cases, you never need to call `renewTokens()` manually.

***

👉 With `oidc-spa`, token renewal is **always correct, efficient, and invisible to you**.

{% tabs %}
{% tab title="Vanilla API" %}
```typescript
import { createOidc } from "oidc-spa/core";

const prOidc = await createOidc({ ... });

// Function to call when we want to renew the token
export function renewTokens(){

   const oidc = await prOidc;
   
   if( !oidc.isUserLoggedIn ){
      throw new Error("Logical error");
   }
   
   oidc.renewTokens(
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
Outside of a React Component:

```typescript
import { getOidc } from "~/oidc";

// Function to call when we want to renew the token
export function renewTokens(){

   const oidc = await getOidc({ assert: "user logged in" });
   
   oidc.renewTokens(
      // Optionally you can pass extra params that will be added 
      // to the body of the POST request to the openid-connect/token endpoint.
      // { extraTokenParams: { electedCustomer: "customer123" } }
      // This parameter can also be provided as parameter to the createOidc
      // function. See: https://github.com/keycloakify/oidc-spa/blob/59b8db7db0b47c84e8f383a86677e88e884887cb/src/oidc.ts#L153-L163
   );

}

// Subscribing to token renewal

getOidc().then(oidc => {
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

```tsx
import { useState, useEffect } from "react";
import { assert } from "tsafe/assert";
import { useOidc, getOidc } from "~/oidc";

export function MyComponent() {
    const { renewTokens } = useOidc({ assert: "user logged in" });
    return (
        <>
            <button onClick={() => renewTokens()}>Rotate tokens</button>
        </>
    );
}
```
{% endtab %}
{% endtabs %}
