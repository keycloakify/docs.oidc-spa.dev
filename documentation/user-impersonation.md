---
description: Enable the admin of your application to login as a given user.
---

# 👨‍🔧 User impersonation

{% hint style="warning" %}
User impersonation should be handled via the authentication server.  \
For example if you are using Keycloak, from the Admin Console you can navigate to&#x20;

users -> action -> impersonate

This will let you use all the apps authenticated by the selected realm authenticated as a given user.  \
\
The approach described in this documentation page is only a workaround in the case where:  \
&#x20;

* The support team that will impersonate users don't have access to the Keycloak Admin Console.
* **And** you [can't host your custom admin app under a sub path of the auth server url](#user-content-fn-1)[^1].
{% endhint %}

Let's say you have a custom admin app that should enable your support team to impersonate your users.  \
Keycloakify enable you to to provide a special query parameter when you redirect a member of your support team to your app from your admin app so that authentication for a given user is automatically enabled.  \
\
This feature is disabled by default, to enable it: &#x20;

{% tabs %}
{% tab title="Vanilla API" %}
```typescript
import { createOidc } from "oidc-spa";

const oidc = await createOidc({
    // ...
    getDoContinueWithImpersonation: async ({ parsedAccessToken })=> {
    
      const doContinue = confirm(`
        WARNING: You are about to impersonate ${parsedAccessToken.email}.
        If you don't understand why you are seeing this message please
        click cancel and contact support.  
        Someone might be trying to trick you.  
      `);
      
      return doContinue;
        
    }
});
```
{% endtab %}

{% tab title="React API" %}
```tsx
import { createReactOidc } from "oidc-spa/react";

export const { OidcProvider, useOidc, getOidc } = createReactOidc({
    // ...
    getDoContinueWithImpersonation: async ({ parsedAccessToken })=> {
    
      const doContinue = confirm(`
        WARNING: You are about to impersonate ${parsedAccessToken.email}.
        If you don't understand why you are seeing this message please
        click cancel and contact support.  
        Someone might be trying to trick you.  
      `);
      
      return doContinue;
        
    }
});

```
{% endtab %}
{% endtabs %}

### Crafting the url for impersonation

Assuming you have called the Keycloak Admin REST API to optain an access token, id token and a refresh token for a given user session, here is how to can craft the redirection url for impersonation:  \
\
(In this snippet we assume that  you have a JavaScript backend but you can adapt it)\


```typescript

const accessToken = "...";
const idToken = "...";
const refreshToken = "...";

const obj = {
    accessToken,
    idToken,
    refreshToken,
};

// NOTE: An array in case you have more than one oidc client instance in your app.
const arr = [obj];
const str = JSON.stringify(arr);
const b64 = btoa(objStr); // to base64

// This is the impersonation url:
const url = `https://your-app.com?oidc-spa_impersonate=${b64}`
```





[^1]: For example let's say you can be in this situation:\
    \- App: company.com\
    \- Keycloak: company.com/auth\
    \- Custom admin: company/admin\
    \
    In this case there is a much better way to implement impersonation come ask about it on [the oidc discord server](https://discord.gg/mJdYJSdcm4).
