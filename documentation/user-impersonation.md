---
description: Enable the admin of your application to login as a given user.
---

# 👨‍🔧 User impersonation



{% hint style="warning" %}
User impersonation should ideally be managed by the authentication server.\
For instance, if you are using Keycloak, you can navigate to the Admin Console, then go to:\
**Users** -> **Action** -> **Impersonate**.\
This allows you to access all applications within the realm as the impersonated user.

The workaround described in this documentation is intended for situations where:

* The support team handling impersonation does not have access to the Keycloak Admin Console.
* Hosting a custom admin app [under a subpath of the authentication server's URL is not an option](#user-content-fn-1)[^1].
{% endhint %}

Imagine you have a custom admin app that allows your support team to impersonate users.\
With oidc-spa, you can include a special query parameter when redirecting a support team member from your admin app to your main app. This will automatically authenticates the support team member as the impersonated user.

By default, this feature is disabled. To enable it:

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

### Crafting the URL for Impersonation

After using the Keycloak API to obtain an access token, ID token, and refresh token for a user session in exchange for your admin token, you can craft the redirection URL for impersonation as follows:

(For this example, we assume you're using a JavaScript backend, but you can easily adapt it to your environment.)

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
const b64 = btoa(str); // to base64

// This is the impersonation url:
const url = `https://your-app.com?oidc-spa_impersonate=${b64}`
```

[^1]: For example let's say you can be in this situation:\
    \- App: company.com\
    \- Keycloak: company.com/auth\
    \- Custom admin: company/admin\
    \
    In this case there is a much better way to implement impersonation come ask about it on [the oidc discord server](https://discord.gg/mJdYJSdcm4).
