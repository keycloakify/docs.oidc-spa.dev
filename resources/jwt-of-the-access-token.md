---
description: And why it's not supposed to be read on the client side.
icon: brackets-curly
---

# JWT Of the Access Token

You might be surprised, or even frustrated, that oidc-spa only provides the decoded ID token and not the decoded access token. This is intentional: the access token is meant to be **opaque** to the client application. It should be used only as an authentication key (e.g., a Bearer token when calling an API). According to the OAuth 2.0 specification, [the access token is not even required to be a JWT](https://datatracker.ietf.org/doc/html/rfc6749#section-1.4):

> The string is usually opaque to the client. \[...] The token may denote an identifier used to retrieve the authorization information or may self-contain the authorization information in a verifiable manner (i.e., a token string consisting of some data and a signature).

The good news is that everything you need is usually found in the ID token. If you notice that certain information appears in the access token but not in the ID token, there are two likely reasons:

1. **Identity server policy** – Your identity provider may have an explicit rule stripping or not including those claims in the ID token. For example, [Keycloak does not include](https://github.com/keycloak/keycloak/issues/14617#issuecomment-1268412474) the `realm_access` claim in the ID token by default.
2. **Schema filtering** – [When using `decodedIdTokenSchema` with Zod](https://docs.oidc-spa.dev/integration-guides/usage#basic-usage), any claims not declared in your schema will be discarded. This can make it seem like the ID token contains fewer claims than it actually does. To see the complete payload, initialize the adapter with `debugLogs: true`, disable `decodedIdTokenSchema`, and check your browser console output.

## Manually decoding the access token

If you absolutely need to introspect the access token, such as when migrating from another library and you cannot modify the IDP's configuration, you can decode it manually using:

{% tabs %}
{% tab title="Vanilla API" %}
```typescript
import { decodeJwt } from 'oidc-spa/decode-jwt';

const decodedAccessToken = decodeJwt(await oidc.getAccessToken());
```
{% endtab %}

{% tab title="React" %}
{% code title="src/oidc.ts" %}
```typescript
import { oidcSpa } from "oidc-spa/react-spa";
import { decodeJwt } from "oidc-spa/decode-jwt";

export const {
    bootstrapOidc,
    getOidc,
    // ...
} = oidcSpa
    .withExpectedDecodedIdTokenShape({ /* ... */ })
    .createUtils();

let decodedAccessToken: Record<string, unknown> | undefined;

getOidc().then(async oidc => {
    if (!oidc.isUserLoggedIn) {
        return;
    }

    const accessToken = await oidc.getAccessToken();

    decodedAccessToken = decodeJwt(accessToken);

    // Using Zod to validate the shape is recommended as well:
    // decodedAccessToken = DecodedAccessTokenSchema.parse(decodeJwt(accessToken));
});

export function getDecodedAccessToken(): Record<string, unknown> | undefined {
    if (decodedAccessToken === undefined) {
        throw new Error("Decoded access token accessed too early. Only use in a component inside <OidcInitializationGate />.");
    }

    return decodedAccessToken;
}
```
{% endcode %}
{% endtab %}
{% endtabs %}
