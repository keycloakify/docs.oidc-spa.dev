# 🔁 Tokens Renewal

You probably don't need to do it. The token refresh is handled automatically for you, however you can manually trigger a token refresh: &#x20;

{% tabs %}
{% tab title="Vanilla API" %}
```typescript
import { createOidc } from "oidc-spa";

const oidc = await createOidc({ ... });

if( oidc.isUserLoggedIn ){
   oidc.renewToken(
      // Optionally you can pass extra params that will be added 
      // to the body of the POST request to the openid-connect/token endpoint.
      // { extraTokenParams: { electedCustomer: "customer123" } }
      // This parameter can also be provided as parameter to the createOidc
      // function. See: https://github.com/keycloakify/oidc-spa/blob/59b8db7db0b47c84e8f383a86677e88e884887cb/src/oidc.ts#L153-L163
   );
}
```
{% endtab %}

{% tab title="React API" %}
```tsx
import { useOidc } from "oidc";
import { useEffect } from "react";

function MyComponent(){

  const { renewTokens } = useOidc({ assertUserLoggedIn: true });
  
  useEffect(()=> {
      renewTokens(
        // Optionally you can pass extra params that will be added 
        // to the body of the POST request to the openid-connect/token endpoint.
        // { extraTokenParams: { electedCustomer: "customer123" } }
        // This parameter can also be provided as parameter to the createReactOidc
        // function. See: https://github.com/keycloakify/oidc-spa/blob/59b8db7db0b47c84e8f383a86677e88e884887cb/src/oidc.ts#L153-L163
      );
  }, []);
  
  return <>...</>;

}
```
{% endtab %}
{% endtabs %}

You can also track when the token are being refreshed:

{% tabs %}
{% tab title="Vanilla API" %}
```typescript
import { createOidc } from "oidc-spa";

const oidc = await createOidc({ ... });

if( !oidc.isUserLoggedIn ){
    oidc.subscribeToTokensChange(() => {
        console.log("Tokens change", oidc.getTokens());
    });
}
```
{% endtab %}

{% tab title="React API" %}
{% code title="src/oidc.ts" %}
```typescript
import { createReactOidc } from "oidc-spa/react";

export const {
    /* ... */
    getOidc
} = createReactOidc({ /* ... */ });

getOidc().then(oidc => {

    if( !oidc.isUserLoggedIn ){
        return;
    }

    oidc.subscribeToTokensChange(() => {
        console.log("Tokens change", oidc.getTokens());
    });

});
```
{% endcode %}

Or directly in your component:

```tsx
import { useOidc } from "./oidc";

export function PotectedPage() {
    const { oidcTokens } = useOidc({ assertUserLoggedIn: true});

    useEffect(()=> {

        console.log("Tokens changed", oidcTokens);

    }, [oidcTokens]);
    
    // ...
}
```
{% endtab %}
{% endtabs %}
