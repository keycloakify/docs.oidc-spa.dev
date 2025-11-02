# Error Handling - No AutoLogin

If you do not have [Auto Login](../../auto-login.md) enabled and there is an error during initialization of oidc-spa, either due to a miss configuration or becuase the server is down, oidc-spa will load your app with the state of the user: not logged in. (oidc.isUserLoggedIn will be false).  \
\
The goal is to enable the user to at least browse the public pages.&#x20;

If the user click on the login button or navigate to a page that requires autentication in this state they will be met with an alert saying:

> Authentication is currently unavailable. Please try again later.

You can of course customize this behavior or decide if you want to show an error:

{% tabs %}
{% tab title="Framework Agnostic" %}
```typescript
import { createOidc } from "oidc-spa/core";

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

{% tab title="TanStack Start" %}
```tsx
import { createOidcComponent } from "@/oidc";
import { useEffect } from "react";

const AuthButtons = createOidcComponent({
    component: () => {
        const { 
            isUserLoggedIn, 
            login, 
            logout, 
            initializationError 
        } = AuthButtons.useOidc();

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
            return <button onClick={() => logout({ redirectTo: "home" })}>Logout</button>;
        }

        return (
            <button
                onClick={() => {
                    if (initializationError) {
                        alert("Can't login now, try again later");
                        return;
                    }

                    login({ ... });
                }}
            >
                Login
            </button>
        );
    }
});

```
{% endtab %}

{% tab title="React SPAs" %}
```tsx
import { useOidc } from "~/oidc";
import { useEffect } from "react";

function AuthButtons() {

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

{% tab title="Angular" %}
<pre class="language-tsx" data-title="src/app/app.ts"><code class="lang-tsx">import { Component } from '@angular/core';
import { RouterLink, RouterLinkActive, RouterOutlet } from '@angular/router';
import { Oidc } from './services/oidc.service';
import { createKeycloakUtils } from 'oidc-spa/keycloak';
import { inject } from '@angular/core';

@Component({
  selector: 'app-root',
  imports: [RouterOutlet, RouterLink, RouterLinkActive],
  templateUrl: './app.html',
})
export class App {
  oidc = inject(Oidc);

  constructor() {
<strong>    if (!this.oidc.isUserLoggedIn &#x26;&#x26; this.oidc.initializationError) {
</strong><strong>      const { initializationError } = this.oidc;
</strong><strong>
</strong><strong>      // This help you discriminate configuration errors
</strong><strong>      // and error due to the server being temporarely down.
</strong><strong>      console.log(initializationError.isAuthServerLikelyDown);
</strong><strong>
</strong><strong>      // This is a debug message that tells you what's wrong
</strong><strong>      // with your configuration and how to fix it.
</strong><strong>      // (this is not something you want to display to the user)
</strong><strong>      console.log(initializationError.message);
</strong>    }
  }

<strong>  login() {
</strong><strong>    if (this.oidc.isUserLoggedIn) {
</strong><strong>      throw new Error('Control flow error: The user is already logged in');
</strong><strong>    }
</strong><strong>
</strong><strong>    if (this.oidc.initializationError) {
</strong><strong>      alert("Can't login now, try again later");
</strong><strong>      return;
</strong><strong>    }
</strong><strong>
</strong><strong>    return this.oidc.login();
</strong><strong>  }
</strong>
  keycloakUtils = createKeycloakUtils({
    issuerUri: this.oidc.issuerUri,
  });
}

</code></pre>
{% endtab %}
{% endtabs %}
