---
title: Enable security defense
---

{% tabs %}
{% tab title="Framework Agnostic" %}
{% code title="src/oidc.ts" %}
```ts
createOidc({
    // ...
    dpop: "auto"
});
```
{% endcode %}
{% endtab %}

{% tab title="React" %}
{% code title="src/oidc.ts" %}
```ts
bootstrapOidc({
    // ...
    dpop: "auto"
});
```
{% endcode %}
{% endtab %}

{% tab title="Angular" %}
{% code title="src/app/app.config.ts" %}
```ts
Oidc.provide({
  // ...
  dpop: "auto"
});
```
{% endcode %}
{% endtab %}
{% endtabs %}
