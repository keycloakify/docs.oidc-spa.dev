# Installation

{% hint style="info" %}
In this guide, we assume that you have an OIDC-enabled authentication server in place, such as Keycloak.&#x20;

If you have not yet set up such a server, please refer to [our guide for instructions on how to provision and configure a Keycloak server](usage-with-keycloak.md).
{% endhint %}

Let's install oidc-spa in your project: &#x20;

{% tabs %}
{% tab title="npm" %}
```bash
npm install --save oidc-spa
```
{% endtab %}

{% tab title="yarn" %}
```bash
yarn add oidc-spa
```
{% endtab %}

{% tab title="pnpm" %}
```bash
pnpm add oidc-spa
```
{% endtab %}

{% tab title="bun" %}
```bash
bun add oidc-spa
```
{% endtab %}
{% endtabs %}

Create the following file in your public directory: &#x20;

{% code title="public/silent-sso.html" %}
```html
<html>
    <body>
        <script>
            parent.postMessage(location.href, location.origin);
        </script>
    </body>
</html>
```
{% endcode %}
