---
description: >-
  The createOidc function is a function that assists in creating an OpenID
  Connect (OIDC) client, which can be utilized in your application for
  authentication purposes.
---

# createOidc function

#### Parameters

`params`: An object containing the following properties:

* `issuerUri` (string): The URI of the OpenID Connect issuer.
* `clientId` (string): The client ID assigned to your application.
* `transformUrlBeforeRedirect` (optional, function): A function that transforms the string URL before redirection.
* `getExtraQueryParams` (optional, function): A function that returns extra query parameters.
* `publicUrl` (optional, string): The public URL of your application, useful when your app is not hosted at the origin of the domain.



### Returns

A Promise that resolves to an object of type `Oidc`, which can be either `Oidc.LoggedIn` or `Oidc.NotLoggedIn`.

**`Oidc.LoggedIn`**

Represents a logged-in state.

* `isUserLoggedIn` (boolean): `true`.
* `renewTokens` (function): A function to renew tokens.
* `getTokens` (function): A function that returns the current tokens.
* `subscribeToTokensChange` (function): A function to subscribe to token changes.
* `logout` (function): A function to log out.

**`Oidc.NotLoggedIn`**

Represents a not-logged-in state.

* `isUserLoggedIn` (boolean): `false`.
* `login` (function): A function to initiate the login process.
  * `doesCurrentHrefRequiresAuth` (boolean): Whether the current href requires authentication.
  * `extraQueryParams` (optional, object): Extra query parameters for login.

### Example

```typescript
import { createOidc } from 'oidc-spa';

const oidc = await createOidc({
  issuerUri: 'https://your-issuer.com',
  clientId: 'your-client-id',
  transformUrlBeforeRedirect: (url) => `${url}&ui_locales=fr`,
  getExtraQueryParams: () => {kc_idp_hint: "idp1" }/* your extra query params */,
  publicUrl: '/my-app',
});
```
