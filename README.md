# 🔩 Installation

{% hint style="info" %}
Before starting be aware that oidc-spa does **not** yet support Next.js projects. &#x20;

If you are using Next the closer alternative is to use [NextAuth.js](https://next-auth.js.org/) (with [the Keycloak adapter](https://next-auth.js.org/providers/keycloak) if you are using Keycloak).  \
You can refer to [the phase two guide](https://phasetwo.io/docs/securing-applications/next/).
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

Create the silent-sso.htm[^1] file in your public directory: &#x20;

{% code title="public/silent-sso.htm" %}
```html
<!doctype html>
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

<summary>Doing without the silent-sso.htm file</summary>

If for some reasons it's not fesable or practical for you to rely on the `silent-sso.html` file it's ok, it will work without it. &#x20;

Just make sure to&#x20;

* Set `publicUrl` to `undefined` when initializing oidc-spa.
* Don't use `logout({ redirectTo: "home" })` but explicitely tell where you want your users to be redirected after logout using `logout({ redirectTo: "specific url", url: "/my-home" })` or use `logout({ redirectTo: "current page" })`.

</details>

[^1]: Yes .htm and not .html.  \
    Some web server such as [serve](https://github.com/vercel/serve#readme) will rewrite `/silent-sso.html` to `/silent-sso` and serve the `index.html` of your distribution.  \
    Using the `.htm` extention prevend any potential issue.
