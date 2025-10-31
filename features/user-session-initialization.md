---
icon: square-caret-up
---

# User Session Initialization

In some cases, you might want to perform some actions when the user login to your app. &#x20;

It might be clearing some storage values, or calling a specific API endpoint.  \
If this action is costly. You might want to avoid doing it over and over again each time the user refresh the page. &#x20;



{% tabs %}
{% tab title="Vanilla API" %}
```typescript
import { createOidc } from "oidc-spa";

const oidc = await createOidc({ /* ... */ });

if (oidc.isUserLoggedIn) {
  if( oidc.isNewBrowerSession ){
     // This is a new visit of the user on your app
     // or the user signed out and signed in again with
     // an other identity.
     
     await api.onboard(); // (Example)
  }else{
     // It was just a page refresh (Ctrl+R)
  }
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
  
  if( oidc.isNewBrowerSession ){
     // This is a new visit of the user on your app
     // or the user signed out and signed in again with
     // an other identity.
     
     await api.onboard(); // (Example)
  }else{
     // It was just a page refresh (Ctrl+R)
  }

});
```
{% endcode %}

You can also do this in your React component (although it's maybe not the best approach)

```tsx
import { useOidc } from "./oidc";
import { useEffect } from "react";

function MyComponent(){

    const { isUserLoggedIn, isNewBrowserSession, backFromAuthServer } = useOidc();
    
    useEffect(()=> {
    
        if( oidc.isNewBrowerSession ){
           // This is a new visit of the user on your app
           // or the user signed out and signed in again with
           // an other identity.
           
           api.onboard(); // (Example)
        }else{
           // It was just a page refresh (Ctrl+R)
        }
    
    }, []);
```
{% endtab %}
{% endtabs %}
