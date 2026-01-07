---
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/oygeayjvIPxroUcp3jt4/features/error-management/error-handling-no-autologin
---

# Error Handling - No AutoLogin

{% hint style="info" %}
This guide only apply if you do **not** have [Auto Login](../auto-login.md) enabled.\
If you have Auto Login enabled follow [this guide instead](error-handling-with-autologin.md).
{% endhint %}

If oidc-spa fails to initialize (because of a **misconfiguration** or because the **authorization server is unavailable**), your app will load with the user state **not logged in** (`oidc.isUserLoggedIn === false`).\
The goal is to let users browse public pages even when authentication cannot start.

If, in this state, the user clicks a “Log in” button or navigates to a page that requires authentication, by default oidc-spa will fire this alert:

> Authentication is currently unavailable. Please try again later.

You can customize this behavior (toast, inline banner, maintenance page, retry, etc.) or surface an error page if that fits your UX.

{% hint style="info" %}
Use `initializationError.isAuthServerLikelyDown` to distinguish a temporary outage from a misconfiguration.\
`initializationError.message` is a **developer-oriented** diagnostic with the likely cause and fix; do **not** show it to end users.
{% endhint %}

{% tabs %}
{% tab title="Framework Agnostic" %}
```typescript
import { createOidc } from "oidc-spa/core";

const oidc = await createOidc(...);

if( !oidc.isUserLoggedIn ){
    // User isn’t logged in: allow the app to render public pages and stop here.
    return;
}

if( oidc.initializationError ){

    // Helps you distinguish a misconfiguration from a temporary auth-server outage.
    console.log(oidc.initializationError.isAuthServerLikelyDown);
    
    // Developer-only diagnostic with likely cause and fix.
    // Do not display this to end users.
    console.log(initializationError.message);
    
    const handleLoginClick = ()=> {
    
        if( oidc.initializationError ){
            // Developer note: keep this user-facing message short and neutral.
            alert("Can't login now, try again later");
            return;
        }
        
        oidc.login(...);
    
    };
}
```
{% endtab %}

{% tab title="React" %}
```tsx
import { useOidc } from "~/oidc";
import { useEffect } from "react";

function AuthButtons() {

    const { isUserLoggedIn, login, logout, initializationError } = useOidc();

    useEffect(() => {
        if (initializationError) {
            // Helps distinguish misconfiguration vs. temporary auth-server outage.
            console.log(initializationError.isAuthServerLikelyDown);
        
            // Developer-only diagnostic with likely cause and fix.
            // Do not display this to end users.
            console.log(initializationError.message);
        }
    }, []);

    if (isUserLoggedIn) {
        return <button onClick={()=> logout({ redirectTo: "home" })}>Logout</button>;
    }

    return (
        <button onClick={() => {

            if (initializationError) {
                // Keep the UX calm and actionable.
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
<pre class="language-tsx" data-title="src/app/app.ts"><code class="lang-tsx">@Component({
  selector: 'app-root',
  templateUrl: './app.html',
})
export class App {
  oidc = inject(Oidc);

  constructor() {
<strong>    if (!this.oidc.isUserLoggedIn &#x26;&#x26; this.oidc.initializationError) {
</strong><strong>      const { initializationError } = this.oidc;
</strong><strong>
</strong><strong>      // Helps distinguish a misconfiguration from a temporary auth-server outage.
</strong><strong>      console.log(initializationError.isAuthServerLikelyDown);
</strong><strong>
</strong><strong>      // Developer-only diagnostic with the likely cause and fix.
</strong><strong>      // Do not display this to end users.
</strong><strong>      console.log(initializationError.message);
</strong>    }
  }

<strong>  login() {
</strong><strong>    if (this.oidc.isUserLoggedIn) {
</strong><strong>      throw new Error('Control flow error: The user is already logged in');
</strong><strong>    }
</strong><strong>
</strong><strong>    if (this.oidc.initializationError) {
</strong><strong>      // Keep the UX calm and actionable.
</strong><strong>      alert("Can't login now, try again later");
</strong><strong>      return;
</strong><strong>    }
</strong><strong>
</strong><strong>    return this.oidc.login();
</strong><strong>  }
</strong>
}
</code></pre>
{% endtab %}
{% endtabs %}
