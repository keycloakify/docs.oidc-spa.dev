---
icon: masks-theater
---

# Mock Modes

{% hint style="info" %}
This is the server-side mock mode.\
If you’re looking for the frontend mock mode, see [the project example for your stack](../example-setups.md).
{% endhint %}

`oidc-spa/server` provides two modes to facilitate backend unit testing.

These modes help you run tests in a reproducible way, without fetching the public key from a real IdP.

## Static identity

In this mode, `oidc-spa/server` ignores the provided token.\
It behaves as if every request comes from a user with the identity you define.

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

## Decode only

{% hint style="danger" %}
WARNING: If you accidentally ship this mode to production, it’s catastrophic.\
Everything will appear to work, but an attacker can impersonate anyone.
{% endhint %}

In this mode, `oidc-spa/server` decodes the access token payload, but skips all cryptographic validation.\
This is useful if you’ve saved tokens for unit tests and want those tests to keep working long after the tokens expire.

```typescript
bootstrapAuth({
    implementation: "mock",
    behavior: "decode only",
});
```
