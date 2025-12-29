---
icon: masks-theater
---

# Mock Modes

{% hint style="info" %}
This is the server side mock mode. If you're looking for the documentation of the mock mode on the frontend refer to [the project example fro your stack](../example-setups.md).
{% endhint %}

oidc-spa/server provides two modes to facilitates unit testing of your backend. &#x20;

Those modes are usefull to run your test in a reproductible way, without having to actually fetch the public key of a real IdP.

## Static identity

This mode will make oidc-spa/server ignore the token that is actually provided and pretend it's a user with a given identity that is making all the request. &#x20;

```typescript
bootstrapAuth({
    implementation: "mock",
    behavior: "use static identity",
    decodedAccessToken_mock: {
        sub: "123",
        name: "John Doe",
        email: "john.doe@gmail.com",
        realm_access: {
            roles: ["realm-admin", "support-staff"]
        }
    }
});
```

## Decode Only

{% hint style="danger" %}
WARNING: If you accidentaly ship to prod with this mode enabled, this is catastophic. Everything will apprear to work as usual but an atacker will be able to impersonate anyone.
{% endhint %}

In this mode, oidc-spa/server will decode the palyload of the access token but skip all cryptographic validation.  \
This mode is usefull for example if you have dumped some token to run unit tests and you want to be able to run those same test even after they are long expired. &#x20;

```typescript
bootstrapAuth({
    implementation: "mock",
    behavior: "decode only",
});
```

