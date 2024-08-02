# 🔐 User Account Management

{% hint style="info" %}
This section is Keycloak specific
{% endhint %}

When your user is logged in, you can provide a link to redirect to Keycloak so they can change their password and accout informations.

{% tabs %}
{% tab title="Vanilla API" %}
```typescript
import { createOidc } from "oidc-spa";

const oidc = await createOidc({ ... });

if( oidc.isUserLoggedIn ){
   oidc.goToAuthServer({
      extraQueryParams: { 
          kc_action: "UPDATE_PASSWORD" 
          //kc_action: "UPDATE_PROFILE"
      }
   });

}
```
{% endtab %}

{% tab title="React API" %}
```tsx

function ProtectedPage() {
    // Here we can safely assume that the user is logged in.
    const { goToAuthServer } = useOidc({ assertUserLoggedIn: true });

    return (
        <button
            onClick={() =>
                goToAuthServer({
                    extraQueryParams: { 
                        kc_action: "UPDATE_PASSWORD",
                        // kc_action: "UPDATE_PROFILE"
                    }
                })
            }
        >
            Change password
            {/* Update my Account informations */}
        </button>
    );
}
```
{% endtab %}
{% endtabs %}
