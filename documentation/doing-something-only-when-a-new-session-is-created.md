# 🔄 Doing Something Only When a New Session is Created

In some cases, you might want to perform some operation to initialize the user's session. This could involve calling a special API endpoint or clearing some cached values in the local storage.\
What you don't want, however, is to run this every time the user refreshes the page or when their session is restored.\
To help you determine if the session should be initialized, you can leverage the `authMethod` property that is available when the user is logged in.

There are three possible values for the `authMethod` property:

1. **"back from auth server"**: The user was redirected to the authentication server's login/registration page and then redirected back to the application. Assuming you are using Keycloak and if you have configured your Keycloak server as suggested in [the configuration guide](../resources/usage-with-keycloak.md), this happens approximately once every 14 days, assuming the user is using the same browser and has not explicitly logged out. Of course, the 14-day session is just a good default if you don't want your user to go through the login process every day, but this is for you to decide. If you implement an [Auto Logout mechanism](auto-logout.md), it will be, of course, much shorter.
2. **"session storage"**: The user's authentication was restored from the browser session storage, typically after a page refresh. As soon as the user closes the tab, the session storage is cleared.
3. **"silent signin"**: The user was authenticated silently using an iframe to check the session with the authentication server. This happens most of the time when the user navigates to your app in a new tab and their session has not expired yet.

{% tabs %}
{% tab title="Vanilla API" %}
```typescript
import { createOidc } from "oidc-spa";

const oidc = await createOidc({ /* ... */ });

if (oidc.isUserLoggedIn && oidc.isAuthMethod === "back from auth server") {
  // Do something related to initialization or clearing cache
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

    if( oidc.authMethod === "back from auth server" ){
        // Do something related to initialization or clearing cache
    }

});
```
{% endcode %}

You can also do this in your React component (although it's maybe not the best approach)

```tsx
import { useOidc } from "./oidc";
import { useEffect } from "react";

function MyComponent(){

    const { isUserLoggedIn, authMethod } = useOidc();
    
    useEffect(()=> {
    
        // Warning! In dev mode, when React Strict Mode is enabled
        // this will be called twice!
        
        if( !isUserLoggedIn ){
            return;
        }
        
        if( authMethod === "back from auth server" ){
           // Do something related to initialization or clearing cache
        }
    
    }, []);

}
```
{% endtab %}
{% endtabs %}

