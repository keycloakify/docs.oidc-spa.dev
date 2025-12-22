---
description: 'RFC 9449: OAuth 2.0 Demonstrating Proof-of-Possession (DPoP)'
icon: receipt
---

# DPoP

Demonstrating Proof of Possesion in a nutshell is a protocol level security defence that essentially makes the access tokens unusable, even if intercepted. &#x20;

It's widely supported, by Keycloak and most modern backend token validation library. &#x20;

oidc-spa enables you to fully transparently enable DPoP without changing a single line of code in your application! &#x20;

## Enabling DPoP

{% hint style="info" %}
The only reason DPoP isn't automatically enabled when supported by the auth server is that some older token validation libraries does not support it yet.  \
But if you use oidc-spa/server on the backend or any modern library to validate tokens you can and should set dpop: "auto"
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

When DPoP is enabled, oidc-spa will automatically update every outgoing request made with fetch or XMLHttpRequest and replace:

{% code title="Request Header" %}
```
Authorization: Bearer <Access Token>
```
{% endcode %}

With:

{% code title="Request Header" %}
```
Authorization: DPoP <Access Token>
DPoP:          <DPoP Proof>
```
{% endcode %}

It will also track the nonce issued by the server in the response header.  <br>

The fetch and XMLHttpRequest interceptors are registered either via the Vite plugin or during the execution of oidcEarlyInit(). It's completely transparent to you. You can forget about DPoP and should still send your requests wtih Authorization: \`Bearer ${accessToken}\`. &#x20;
