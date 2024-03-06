---
description: And why it's not supposed to be read on the client side.
---

# 🗝️ JWT Of the Access Token

You might be surprised or even frustrated by the fact that oidc-spa only provides the decoded id token but not the decoded access token. &#x20;

Infact the access token is supposed to be opaque for the client application is to be used only as an autentication key such as a Barrer token for an API. &#x20;

As per the OIDC standard the access token is not even required to be a JWT! &#x20;

But worry not, everything that you keed is probably in the id token, if there is something missing in your id token that is present in your access token there is an explicit policy on your identity server in place that strips this information out.   \
Zod is stripping out all the claims that are not specified in the shema. This might have led you to beleve that there is less informations in the id token than what actually is. &#x20;

{% embed url="https://github.com/keycloakify/oidc-spa/blob/a5dbcca428721c4ea4ed1fd36dc07e80fc9d2cb0/examples/tanstack-router/src/oidc.tsx#L28-L38" %}

If, however, you still want to access the infos in the access token you can do it with: &#x20;

{% tabs %}
{% tab title="Vanilla API" %}
```typescript
import { decodeJwt } from "oidc-spa/tools/decodeJwt";

const decodedAccessToken = decodeJwt(oidc.getTokens().accessToken);
```
{% endtab %}

{% tab title="Second Tab" %}
```tsx
import { decodeJwt } from "oidc-spa/tools/decodeJwt";

const { oidcTokens } = useOidc();

const decodedAccessToken = decodeJwt(oidcTokens.accessToken);
```
{% endtab %}
{% endtabs %}
