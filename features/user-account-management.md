---
icon: file-user
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/UhNOMoIddws1XoAnT5Nn/features/user-account-management
---

# User Account Management

## Redirecting to your IdP's account managment page

<figure><img src="../.gitbook/assets/image (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

IdP always provide a user account page that let users, update their password, account information, manage their session.  \
If you are using Keycloak you can generate the link to the Account Console with:

{% tabs %}
{% tab title="Framework Agnostic" %}
```typescript
import { createKeycloakUtils } from "oidc-spa/keycloak";

const keycloakUtils = createKeycloakUtils({ issuerUri: oidc.issuerUri });

const accountLinkUrl = keycloakUtils.getAccountUrl({
    clientId: oidc.clientId,
    validRedirectUri: oidc.validRedirectUri,
    locale: "en" // Optional
});
```
{% endtab %}

{% tab title="React" %}
```typescript
const { issuerUri, clientId, validRedirectUri } = useOidc();

const keycloakUtils = createKeycloakUtils({ issuerUri });

const accountLinkUrl = keycloakUtils.getAccountUrl({
    clientId,
    validRedirectUri,
    locale: "en" // Optional
});
```
{% endtab %}

{% tab title="Angular" %}
{% code title="src/app/app.ts" %}
```typescript
import { Oidc } from './services/oidc.service';

@Component({
  selector: 'app-root',
  templateUrl: './app.html',
})
export class App {
  oidc = inject(Oidc);
  keycloakUtils = createKeycloakUtils({
    issuerUri: this.oidc.issuerUri,
  });

  accountUrl = this.keycloakUtils.getAccountUrl({
    clientId: this.oidc.clientId,
    validRedirectUri: this.oidc.validRedirectUri,
    locale: "en" // Optional
  })
}
```
{% endcode %}
{% endtab %}
{% endtabs %}

## Direct Link to Specific Actions

{% hint style="info" %}
In this section we assume you are using Keycloak. If you are using another authentication server you'll have to addapt the `queryParameter` provided.
{% endhint %}

<figure><img src="../.gitbook/assets/image (2) (1).png" alt=""><figcaption></figcaption></figure>

There is thee main actions:

* **UPDATE\_PASSWORD**: Enables the user to change their password.
* **UPDATE\_PROFILE**: Enable the user to edit teir account information such as first name, last name, email, and any additional user profile attribute that  you might have configured on your Keycloak server.
* **delete\_account**: (In lower case): This enables the user to delete he's account. You must enable it manually on your Keycloak server Admin console. See [Keycloak Configuration Guide](../providers-configuration/keycloak.md).

Let's, as an example, how you would implement an update password button:

{% tabs %}
{% tab title="Vanilla API" %}
```typescript
import { createOidc } from "oidc-spa";
import { parseKeycloakIssuerUri } from "oidc-spa/tools/parseKeycloakIssuerUri";

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
      oidc.backFromAuthServer?.extraQueryParams.kc_action === "UPDATE_PASSWORD"
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

// Url for redirecting users to the keycloak account console.
const keycloakAccountUrl = parseKeycloakIssuerUri(oidc.params.issuerUri)
   .getAccountUrl({ 
       clientId: params.clientId,
       backToAppFromAccountUrl: `${location.href}${import.meta.env.BASE_URL}`
    });
        
```
{% endtab %}

{% tab title="React" %}
```tsx
import { useOidc } from "@/oidc";

function ProtectedPage() {
    // Here we can safely assume that the user is logged in.
    const { goToAuthServer, backFromAuthServer, params } = useOidc({ assert: "user logged in" });
    
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

{% tab title="Angular" %}
```typescript
updatePassword = ()=> this.oidc.goToAuthServer({
    extraQueryParams: { kc_action: "UPDATE_PASSWORD" }
});
```

```angular-html
@if( oidc.backFromAuthServer?.extraQueryParams.kc_action === "UPDATE_PASSWORD" ){          
@if ( oidc.backFromAuthServer.result.kc_action_status === "success" ){
<p>Password successfully updated</p>
} @else {
<P>Password unchanged</p>
}
}
```
{% endtab %}
{% endtabs %}
