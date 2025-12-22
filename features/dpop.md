---
description: 'RFC 9449: OAuth 2.0 Demonstrating Proof-of-Possession'
icon: receipt
---

# DPoP

[Demonstrating Proof-of-Possesion](https://auth0.com/docs/secure/sender-constraining/demonstrating-proof-of-possession-dpop) is a protocol level security defense that make it so that access token are not suficient on their own to access resource server. It makes access tokens much less sensible and afford you the peice of mind to know that if they ever leak, concequence are very limited. &#x20;

It's supported by Keycloak and many other IdPs. &#x20;

## Enabling DPoP

{% hint style="info" %}
The only reason DPoP isn't automatically enabled in oidc-spa is that some older token validation libraries might not support it yet.  \
If you use oidc-spa/server or another modern library to validate tokens you can and should set dpop: "auto"
{% endhint %}

{% tabs %}
{% tab title="Framework Agnostic" %}
{% code title="src/oidc.ts" %}
```typescript
createOidc({ 
    // ...
    dpop: "auto" // Enabled if supported by the Auth server you are using.
    /* OR: 
    dpop: "disable" // Default
    dpop: "enabled" // oidc-spa will refuse to start if the Auth server does not support it.
    */
});
```
{% endcode %}
{% endtab %}

{% tab title="React" %}
{% code title="src/oidc.ts" %}
```typescript
bootstrapOidc({
    // ...
    dpop: "auto" // Enabled if supported by the Auth server you are using.
    /* OR: 
    dpop: "disable" // Default
    dpop: "enabled" // oidc-spa will refuse to start if the Auth server does not support it.
    */
});
```
{% endcode %}
{% endtab %}

{% tab title="Angular" %}
{% code title="src/app/app.config.ts" %}
```typescript
Oidc.provide({
  // ...
  dpop: "auto" // Enabled if supported by the Auth server you are using.
  /* OR: 
  dpop: "disable" // Default
  dpop: "enabled" // oidc-spa will refuse to start if the Auth server does not support it.
  */
})
```
{% endcode %}
{% endtab %}
{% endtabs %}

## How it works

When DPoP is enabled, oidc-spa will automatically upgrade any outgoing authed request your app sends: &#x20;

{% code title="Request Header - Set by you" %}
```
Authorization: Bearer <Access Token>
```
{% endcode %}

{% code title="Request Header - Actually goes out" %}
```
Authorization: DPoP <Access Token>
DPoP:          <DPoP Proof>
```
{% endcode %}

It will also track DPoP nonce that might be issued by resource servers in response headers. &#x20;

To accheve that transparently, oidc-spa register a fetch() and XMLHttpRequest interceptor via the Vite plugin or during the execution of oidcEarlyInit(). It's completely transparent to you. You can forget about DPoP and just continue sending your requests like you used to wtih `` Authorization: `Bearer ${accessToken}` ``. &#x20;
