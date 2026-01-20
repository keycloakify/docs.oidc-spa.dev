---
title: entra id config
---

{% tabs %}
{% tab title="Framwork Agnostic" %}
{% code title="src/oidc.ts" %}
```typescript
// Directory (tenant) ID:
const directoryId = "XXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX";
// Application (client) ID:
const clientId = "XXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX";
// Application ID URI: (Of the API!)
const applicationIdUri_api= "api://my-app-api/access_as_user";

createOidc({
    issuerUri: `https://login.microsoftonline.com/${directoryId}/v2.0`,
    clientId,
    scopes: ["profile", applicationIdUri_api],
    // ...
});
```
{% endcode %}
{% endtab %}

{% tab title="React" %}
{% code title="src/oidc.ts" %}
```typescript
// Directory (tenant) ID:
const directoryId = "XXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX";
// Application (client) ID:
const clientId = "XXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX";
// Application ID URI: (Of the API!)
const applicationIdUri_api= "api://my-app-api/access_as_user";

bootstrapOidc({
    issuerUri: `https://login.microsoftonline.com/${directoryId}/v2.0`,
    clientId,
    scopes: ["profile", applicationIdUri_api],
    // ...
});
```
{% endcode %}
{% endtab %}

{% tab title="Angular" %}
{% code title="src/app/app.config.ts" %}
```typescript
// Directory (tenant) ID:
const directoryId = "XXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX";
// Application (client) ID:
const clientId = "XXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX";
// Application ID URI: (Of the API!)
const applicationIdUri_api= "api://my-app-api/access_as_user";

Oidc.provide({
    issuerUri: `https://login.microsoftonline.com/${directoryId}/v2.0`,
    clientId,
    scopes: ["profile", applicationIdUri_api],
})
```
{% endcode %}
{% endtab %}
{% endtabs %}
