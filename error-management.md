---
description: Gracefully handle authentication issues
icon: message-exclamation
---

# Error Management

What happens if the OIDC server is down, or if your OIDC server isn't properly configured?

By default, [if you don't have autoLogin enabled](#user-content-fn-1)[^1], when there is an error with the OIDC initialization your website will load with the user unauthenticated. &#x20;

This allows the user to at least access parts of the application that do not require authentication. When the user clicks on the login button (triggering the `login()` function), a browser alert is displayed, indicating that authentication is currently unavailable, and no further action is taken.&#x20;

You can customize this behavior. An `initializationError` object is present on the `oidc` object if an error occurred.

{% tabs %}
{% tab title="Vanilla API" %}
```typescript
import { createOidc } from "oidc-spa";

const oidc = await createOidc(...);

if( !oidc.isUserLoggedIn ){
    // If the used is logged in we had no initialization error.
    return;
}

if( oidc.initializationError ){

    // This help you discriminate configuration errors
    // and error due to the server being temporarely down.
    console.log(oidc.initializationError.isAuthServerLikelyDown);
    
    const handleLoginClick = ()=> {
    
        if( oidc.initializationError ){
            alert(`Can't login now, try again later ${oidc.initializationError.message}`);
            return;
        }
        
        oidc.login(...);
    
    };
}
```
{% endtab %}

{% tab title="React API" %}
```tsx
import { useOidc } from "oidc";
import { useEffect } from "react";

function LoginButton() {

    const { isUserLoggedIn, login, logout, initializationError } = useOidc();

    useEffect(() => {
        if (initializationError) {
            // This help you discriminate configuration errors
            // and error due to the server being temporarely down.
            console.log(initializationError.isAuthServerLikelyDown);
        
            // This is a debug message that tells you what's wrong
            // with your configuration and how to fix it.  
            // (this is not something you want to display to the user)
            console.log(initializationError.message);
        }
    }, []);

    if (isUserLoggedIn) {
        return <button onClick={()=> logout({ redirectTo: "home" })}>Logout</button>;
    }

    return (
        <button onClick={() => {

            if (initializationError) {
                alert("Can't login now, try again later")
                return;
            }

            login({ ... });

        }}>
            Login
        </button>
    );

}
```
{% endtab %}
{% endtabs %}

[^1]: If you do have autoLogin enabled [checkout how to handle errors](auto-login.md).
