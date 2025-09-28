---
icon: angular
---

# Angular

## Basic Example

```bash
npx degit https://github.com/keycloakify/oidc-spa/examples/angular oidc-spa-angular
cd oidc-spa-angular
npm install
npm run start
```

{% embed url="https://github.com/keycloakify/oidc-spa/tree/main/examples/angular" %}

## Advanced example

Live here: [https://example-angular.oidc-spa.dev](https://example-angular.oidc-spa.dev/)

This setup show you how you can:&#x20;

* Early rendering of public pages before oidc has finished initializing.
* Mock implementation of the adapter.
* Fetching the initialization parameter remotly.
* Protecting groupes based on roles.
* Validating the shape of the access token.

```bash
npx degit https://github.com/keycloakify/oidc-spa/examples/angular-kitchensink oidc-spa-angular-kitchensink
cd oidc-spa-angular-kitchensink
npm install
npm run start
```

{% embed url="https://github.com/keycloakify/oidc-spa/tree/main/examples/angular-kitchensink" %}
