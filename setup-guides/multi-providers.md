---
icon: split
hidden: true
---

# Multi Providers - Login with Google or Microsoft

{% hint style="warning" %}
This approach is discouraged.

Instead of managing multiple authentication providers within your application, consider using [Keycloak](https://www.keycloak.org/) or [Auth0](https://auth0.com/) to streamline and centralize identity provider integration.
{% endhint %}

The example setup is live here: [https://example-multi-providers.oidc-spa.dev/](https://example-multi-providers.oidc-spa.dev)

Run it locally with: &#x20;

```bash
npx degit https://github.com/keycloakify/oidc-spa/examples/multi-providers oidc-spa-multi-providers
cd oidc-spa-multi-providers
# NOTE: With this example only internat Insee Microsoft account can sign in.
# You can however signin with your personal Google Account.
cp .env.local.sample .env.local
yarn
yarn dev
```

{% embed url="https://github.com/keycloakify/oidc-spa/tree/main/examples/multi-providers" %}

