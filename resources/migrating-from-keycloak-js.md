---
icon: arrow-up-to-dotted-line
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/oygeayjvIPxroUcp3jt4/resources/migrating-from-keycloak-js
---

# Migrating from Keycloak-js

{% hint style="warning" %}
Still under construction, please wait a little before using. &#x20;
{% endhint %}

{% code title="package.json" %}
```diff
 {
     dependencies: {
-        "keycloak-js": "...",
+        "oidc-spa": "..."
     }
 }
```
{% endcode %}

{% tabs %}
{% tab title="Vite SPAs" %}
In Vite apps, this is done through a Vite Plugin (If you'd rather avoid using the Vite plugin checkout the Other SPAs tab).

<pre class="language-typescript" data-title="vite.config.ts"><code class="lang-typescript">import { defineConfig } from "vite";
<strong>import { oidcSpa } from "oidc-spa/vite-plugin";
</strong>
export default defineConfig({
    plugins: [
        // ...
<strong>        oidcSpa({
</strong><strong>            freezeFetch: true,
</strong><strong>            freezeXMLHttpRequest: true,
</strong><strong>            freezeWebSocket: true
</strong><strong>        })
</strong>    ]
});
</code></pre>
{% endtab %}

{% tab title="Other SPAs" %}
First rename your entry point file from `main.tsx` (or `main.ts` or whatever it is) to `main.lazy.tsx`

```bash
mv src/main.ts src/main.lazy.ts
```

Then create a new `index.tsx` file:

{% code title="src/index.tsx" %}
```typescript
import { oidcEarlyInit } from "oidc-spa/entrypoint";

const { shouldLoadApp } = oidcEarlyInit({
    freezeFetch: true,
    freezeXMLHttpRequest: true,
    freezeWebSocket: true,
    BASE_URL: "/" // The path where your app is hosted, can also be provided later to createOidc()
});

if (shouldLoadApp) {
    // Note: Deferring the main app import adds a few milliseconds to cold start,
    // but dramatically speeds up auth. Overall, it's a net win.
    import("./index.lazy");
}
```
{% endcode %}

If you don't have a precise entrypoint that you can simply override, just call oidcEarlyInit as soon as possible and try canceling as much work as possible when `shouldLoadApp` is false.
{% endtab %}
{% endtabs %}

Then: &#x20;

```diff
- import Keycloak from "keycloak-js";
+ import { Keycloak } from "oidc-spa/keycloak-js";
```

Yes really it's that simple. &#x20;
