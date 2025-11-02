---
description: Gracefully handle authentication issues
icon: message-exclamation
---

# Debug and Error Handling

What happens if the OIDC server is down, or if your OIDC server isn't properly configured?

## Debug your configuration in devlopement

oidc-spa can help you debug your setup, you might want to enable the debug logs:

{% tabs %}
{% tab title="Framework Agnostic" %}
{% code title="src/oidc.ts" %}
```typescript
createOidc({ 
    // ...
    debugLogs: true 
});
```
{% endcode %}
{% endtab %}

{% tab title="React" %}
{% code title="src/oidc.ts" %}
```typescript
bootstrapOidc({
    // ...
    debugLogs: true
});
```
{% endcode %}
{% endtab %}

{% tab title="Angular" %}
{% code title="src/app/app.config.ts" %}
```typescript
Oidc.provide({
  // ...
  debugLogs: true
})
```
{% endcode %}
{% endtab %}
{% endtabs %}

And also check "Preseve Log" in the console option so that important info aren't flushed out by navigations.

For example, in the following picture we can deduce from what oidc-spa is saying that we forgot to add `http://localhost:3000/` to the list of Valid Redirect URIs:

<figure><img src=".gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

***

## Gracefully Handling Errors in Production

Error handling is different depending of if you have [Auto Login](auto-login.md) enabled or not.

{% content-ref url="features/error-management/error-handling-no-autologin.md" %}
[error-handling-no-autologin.md](features/error-management/error-handling-no-autologin.md)
{% endcontent-ref %}

{% content-ref url="features/error-management/error-handling-with-autologin.md" %}
[error-handling-with-autologin.md](features/error-management/error-handling-with-autologin.md)
{% endcontent-ref %}
