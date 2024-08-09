# ❗ Error Management

What happens if the OIDC server is down, or if the server indicates that your client configuration is not valid?&#x20;

By default, [if you don't have `isAuthRequiredOnEveryPages` set to `true`](#user-content-fn-1)[^1], when there is an error with the OIDC initialization your website will load with the user unauthenticated. &#x20;

This allows the user to access parts of the application that do not require authentication. When the user clicks on the login button (triggering the `login()` function), a browser alert is displayed, indicating that authentication is currently unavailable, and no further action is taken.&#x20;

You can customize this behavior. An `initializationError` object is present on the OIDC object if an error occurred.

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

    // initializationError.type can be either:
    // - "server down"
    // - "bad configuration"
    // - "unknown" 
    console.log(oidc.initializationError.type);
    
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

    const { isUserLoggedIn, login, initializationError } = useOidc();

    useEffect(() => {
        if (!initializationError) {
            return;
        }

        console.warn("OIDC initialization error");
        switch (initializationError.type) {
            case "bad configuration":
                console.warn("The identity server and/or the client is misconfigured");
                break;
            case "server down":
                console.warn("The identity server is down");
                break;
            case "unknown":
                console.warn("An unknown error occurred");
                break;
        }
    }, []);

    if (isUserLoggedIn) {
        return null;
    }

    return (
        <button onClick={() => {

            if (initializationError) {
                alert(`Can't login now, try again later: ${
                    initializationError.message
                }`);
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

Please note that due to browser security policies, it is impossible to distinguish whether the network is very slow or down, or if the OIDC server has rejected the configuration.

Consequently, one might encounter an error of type `"bad configuration"` on a slow 3G network, for example. &#x20;

However, the timeout duration is automatically adjusted based on the speed of the internet connection of the user, which should prevent this issue from occurring. &#x20;

[^1]: If you do, the error management is a bit different since we can't let the user navigates on the pages that do not requires authentication because every pages requires authentication. In concequence you have to provide an error fallback component. [See Authentication required on every pages](globally-enforce-authentication.md).
