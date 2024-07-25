# 🔐 Update password

{% hint style="info" %}
This is a feature specific to Keycloak
{% endhint %}

When your user is logged in, you can provide a link to redirect to Keycloak so they can change their password. &#x20;

{% tabs %}
{% tab title="Vanilla API" %}
```typescript
import { createOidc } from "oidc-spa";

const oidc = await createOidc({ ... });

if( oidc.isUserLoggedIn ){
   oidc.goToAuthServer({
      extraQueryParams: { "kc_action": "UPDATE_PASSWORD" }
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
                    extraQueryParams: { "kc_action": "UPDATE_PASSWORD" }
                })
            }
        >
            Change password
        </button>
    );
}
```
{% endtab %}
{% endtabs %}
