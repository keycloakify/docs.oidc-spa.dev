---
title: entra id config
---

{% tabs %}
{% tab title="Framwork Agnostic" %}
{% code title="src/oidc.ts" %}
```typescript
// Directory (tenant) ID:
const DIRECTORY_ID = "XXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX";
// Application (client) ID:
const CLIENT_ID = "XXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX";
// Application ID URI: (Of the API!)
const SCOPE_FOR_API= "api://my-app-api/access_as_user";

createOidc({
    issuerUri: `https://login.microsoftonline.com/${DIRECTORY_ID}/v2.0`,
    clientId: CLIENT_ID,
    scopes: ["profile", SCOPE_FOR_API],
    // ...
});
```
{% endcode %}
{% endtab %}

{% tab title="React" %}
{% code title="src/oidc.ts" %}
```typescript
// Directory (tenant) ID:
const DIRECTORY_ID = "XXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX";
// Application (client) ID:
const CLIENT_ID = "XXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX";
// Application ID URI: (Of the API!)
const SCOPE_FOR_API= "api://my-app-api/access_as_user";

bootstrapOidc({
    issuerUri: `https://login.microsoftonline.com/${DIRECTORY_ID}/v2.0`,
    clientId: CLIENT_ID,
    scopes: ["profile", SCOPE_FOR_API],
    // ...
});
```
{% endcode %}
{% endtab %}

{% tab title="Angular" %}
{% code title="src/app/app.config.ts" %}
```typescript
// Directory (tenant) ID:
const DIRECTORY_ID = "XXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX";
// Application (client) ID:
const CLIENT_ID = "XXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX";
// Application ID URI: (Of the API!)
const SCOPE_FOR_API= "api://my-app-api/access_as_user";

Oidc.provide({
    issuerUri: `https://login.microsoftonline.com/${DIRECTORY_ID}/v2.0`,
    clientId: CLIENT_ID,
    scopes: ["profile", SCOPE_FOR_API],
})
```
{% endcode %}
{% endtab %}
{% endtabs %}
