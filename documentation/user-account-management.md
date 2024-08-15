# 🔐 User Account Management

{% hint style="info" %}
In this section we assume the reader is using Keycloak. If you are using another kind of authentication server you'll have to addapt the queryParameter provided.
{% endhint %}

When your user is logged in, you can provide a link to redirect to Keycloak so they can manage their account.

There is thee main actions:

* **UPDATE\_PASSWORD**: Enables the user to change their password.
* **UPDATE\_PROFILE**: Enable the user to edit teir account information such as first name, last name, email, and any additional user profile attribute that  you might have configured on your Keycloak server.
* **delete\_account**: (In lower case): This enables the user to delete he's account. You must enable it manually on your Keycloak server Admin console. See [Keycloak Configuration Guide](../resources/usage-with-keycloak.md).

Let's, as an example, how you would implement an update password button:

{% tabs %}
{% tab title="Vanilla API" %}
```typescript
import { createOidc } from "oidc-spa";

const oidc = await createOidc({ ... });

if( oidc.isUserLoggedIn ){

   // Function to invoke when the user click on your "change my password" button.
   const updatePassword = ()=>
      oidc.goToAuthServer({
         extraQueryParams: { 
             kc_action: "UPDATE_PASSWORD" 
         }
      });
   // NOTE: This is optional, it enables you to display a feedback message
   // when the user is redirected back to your application after completing
   // or canceling the action.
   if( 
      oidc.authMethod === "back from auth server" &&
      oidc.backFromAuthServer.extraQueryParams.kc_action === "UPDATE_PASSWORD"
   ){
      switch(oidc.backFromAuthServer.result.kc_action_status){
          case "canceled": 
             alert("You password was not updated");
             break;
          case "success":
             alert("Your password has been updated successfuly");
             break;
      }
   }
}
```
{% endtab %}

{% tab title="React API" %}
```tsx
function ProtectedPage() {
    // Here we can safely assume that the user is logged in.
    const { goToAuthServer, backFromAuthServer } = useOidc({ assertUserLoggedIn: true });

    return (
        <>
            <button
                onClick={() =>
                    goToAuthServer({
                        extraQueryParams: { kc_action: "UPDATE_PASSWORD" }
                    })
                }
            >
                Change password
            </button>
            {/* 
            Optionally you can display a feedback message to the user when they
            are redirected back to the app after completing or canceling the
            action.
            */}
            {backFromAuthServer?.extraQueryParams.kc_action === "UPDATE_PASSWORD" && (
                <p>
                    {(()=>{
                        switch(backFromAuthServer.result.kc_action_status){
                            case "success":
                                return "Password successfully updated";
                            case "cancelled":
                                return "Password unchanged";
                        }
                    })()}
                </p>
            )}
        </>
    );
}

```
{% endtab %}
{% endtabs %}
