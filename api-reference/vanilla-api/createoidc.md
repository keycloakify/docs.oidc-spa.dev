---
description: >-
  The createOidc() function is a function that assists in creating an OpenID
  Connect (OIDC) client, which can be utilized in your application for
  authentication purposes.
---

# createOidc()

### Parameters

`params`: An object containing the following properties:

* `issuerUri: string` The URI of the OpenID Connect issuer.
* `clientId: string` The client ID assigned to your application.
* `transformUrlBeforeRedirect?: (url:string) => string`  (Optional), A function that transforms the string URL before redirection.
* `getExtraQueryParams?: () => Record<string,string>` (Optional), A function that returns extra query parameters.
* `publicUrl?: string`  (Optional), The public URL of your application, useful when your app is not hosted at the origin of the domain.



### Returns

A Promise that resolves to an object of type `Oidc`, which can be either `Oidc.LoggedIn` or `Oidc.NotLoggedIn`.

**`Oidc.LoggedIn`**

Represents a logged-in state.

* `isUserLoggedIn`: `true`.
* `renewTokens: () => Promise<void>` A function to renew tokens.
*   `getTokens: () => Tokens`: A function that returns the current tokens.

    ```typescript
    type Tokens = {
            accessToken: string;
            accessTokenExpirationTime: number;
            idToken: string;
            refreshToken: string;
            refreshTokenExpirationTime: number;
        };
    ```
* `subscribeToTokensChange` A function to subscribe to token changes.
  * Params&#x20;
    * `onTokenChange: () => void` A function to execute when token changes
  * Returns
    * `unsubscribe: () => void`
* `logout` A function to log out.
  * Params
    * `redirectTo: "home" | "current page"` to be redirected after logout to home or current page
    * **OR** `redirectTo: "specific url", url: string` to be redirected to specific url
  * Returns
    * `Promise<never>`

**`Oidc.NotLoggedIn`**

Represents a not-logged-in state.

* `isUserLoggedIn`: `false`.
* `login`A function to initiate the login process.
  * Params
    * `doesCurrentHrefRequiresAuth` (boolean): Whether the current href requires authentication.
    * `extraQueryParams` (optional, object): Extra query parameters for login.
  * Returns&#x20;
    * `Promise<never>`This promise never resolves because we are redirected to oidc web service&#x20;

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
