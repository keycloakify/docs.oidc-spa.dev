---
description: Gracefully handle authentication issues
icon: message-exclamation
metaLinks:
  alternates:
    - https://app.gitbook.com/s/UhNOMoIddws1XoAnT5Nn/features/error-management
---

# Debug and Error Handling

What happens if your OIDC server is down or misconfigured?\
This guide explains how to debug your setup during development and handle errors gracefully in production.

***

## Debugging in Development

To better understand what’s going on under the hood, enable debug logs in your configuration.\
This will print detailed information to your browser console about OIDC initialization, token validation, and redirects.

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
});
```
{% endcode %}
{% endtab %}
{% endtabs %}

Once enabled, make sure to check **"Preserve Log"** in your browser’s console options so the logs aren’t cleared during redirects.

Here’s a common example:\
If you see a message like this in the console, it usually means your **Valid Redirect URIs** list in your IdP configuration is incomplete:

<figure><img src="../../.gitbook/assets/image (3).png" alt="Console showing missing redirect URI error"><figcaption></figcaption></figure>

In this case, simply add `http://localhost:3000/` (or the appropriate URL for your environment) to your list of valid redirect URIs in the IdP settings.

***

## Gracefully Handling Errors in Production

{% tabs %}
{% tab title="My App doesn't have AutoLogin enabled" %}
{% content-ref url="error-handling-no-autologin.md" %}
[error-handling-no-autologin.md](error-handling-no-autologin.md)
{% endcontent-ref %}
{% endtab %}

{% tab title="My App has AutoLogin enabled" %}
{% content-ref url="error-handling-with-autologin.md" %}
[error-handling-with-autologin.md](error-handling-with-autologin.md)
{% endcontent-ref %}
{% endtab %}
{% endtabs %}
