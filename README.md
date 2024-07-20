# 🔩 Installation

{% hint style="info" %}
Before starting: This is **not** a library for Next.js projects. &#x20;

If you are using Next the closer alternative is to use [NextAuth.js](https://next-auth.js.org/) (with [the Keycloak adapter](https://next-auth.js.org/providers/keycloak) if you are using Keycloak). &#x20;
{% endhint %}

If you're having issues don't hesitate to [reach out on Discord](https://discord.gg/mJdYJSdcm4)!

Let's install [oidc-spa](https://github.com/keycloakify/oidc-spa) in your project: &#x20;

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

<details>

<summary>Doing without the silent-sso.html file</summary>

If for some reasons it's not fesable or practical for you to rely on the `silent-sso.html` file it's ok, it will work without it. &#x20;

Just make sure to&#x20;

* Set `publicUrl` to `undefined` when initializing oidc-spa.
* Don't use `logout({ redirectTo: "home" })` but explicitely tell where you want your users to be redirected after logout using `logout({ redirectTo: "specific url", url: "/my-home" })` or use `logout({ redirectTo: "current page" })`.

</details>
