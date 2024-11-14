# 👮 Disabeling token persistance

By default, oidc-spa saves tokens in session storage, allowing users to skip [the silent sign-in process](#user-content-fn-1)[^1] when they reload your app (CTRL + R). However, token persistence can be disabled, and everything will still function as expected. The only difference is that reloading the app may take slightly longer.

{% tabs %}
{% tab title="Vanilla API" %}
```typescript
import { createOidc } from "oidc-spa";

const oidc = await createOidc({
  // ...
  doDisableTokenPersistence: true
});
```
{% endtab %}

{% tab title="React API" %}
```typescript
import { createReactOidc } from "oidc-spa/react";

export const {
    OidcProvider,
    useOidc
} = createReactOidc({
    // ...
    doDisableTokenPersistence: true
})
```
{% endtab %}
{% endtabs %}

[^1]: The silent sign-in process opens an iframe to the authentication server (e.g., Keycloak) to restore the session using HTTP-only cookies. Depending on the response time of your authentication server, this process can be nearly instant or may take several seconds.
