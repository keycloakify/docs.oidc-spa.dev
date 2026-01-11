---
title: Setup option
---

{% tabs %}
{% tab title="Vite Plugin" %}
If you're in a Vite project, the recomended approach is to use oidc-spa's Vite plugin.

<pre class="language-typescript" data-title="vite.config.ts"><code class="lang-typescript">import { defineConfig } from "vite";
<strong>import { oidcSpa } from "oidc-spa/vite-plugin";
</strong>
export default defineConfig({
    plugins: [
        // ...
<strong>        oidcSpa()
</strong>    ]
});
</code></pre>
{% endtab %}

{% tab title="Manual - Recommended" %}
Pick this approach if:

* You're not in a Vite project and
* Your app has a single client entrypoint.

***

Let's assume your app entrypoint is `src/main.ts`.

First, rename it to `src/main.lazy.ts`.

```bash
mv src/main.ts src/main.lazy.ts
```

Then create a new `src/main.ts` file:

{% code title="src/main.ts" %}
```typescript
import { oidcEarlyInit } from "oidc-spa/entrypoint";

const { shouldLoadApp } = oidcEarlyInit({
    BASE_URL: "/" // The path where your app is hosted
                  // If applicable you should use `process.env.PUBLIC_URL`
                  // or `import.meta.env.BASE_URL`.
                  // This is not an option. There's only one good answer.
});

if (shouldLoadApp) {
    // Note: Deferring the main app import adds a few milliseconds to cold start,
    // but dramatically speeds up auth. Overall, it's a net win.
    import("./main.lazy");
}
```
{% endcode %}
{% endtab %}

{% tab title="Manual - Easy" %}
If you’re not using Vite and you can’t edit your app’s entry file, run `oidcEarlyInit()` in the same module where you call `createOidc()`.

Note however that implementing this option [dowgrade the security posture of your app](../../security-features/overview.md#how-oidc-spa-achieves-this-in-a-nutshell) compared to the two other approaches and, in some instances, might conflict with your client side routing library.

<pre class="language-typescript" data-title="src/oidc.ts"><code class="lang-typescript">import { 
<strong>   oidcEarlyInit, 
</strong>   createOidc 
} from "oidc-spa/core";

// Should run as early as possible.  
<strong>oidcEarlyInit({ 
</strong><strong>   BASE_URL: "/" // The path where your app is hosted
</strong><strong>                 // If applicable you should use `process.env.PUBLIC_URL`
</strong><strong>                 // or `import.meta.env.BASE_URL`.
</strong><strong>                 // This is not an option. There's only one good answer.
</strong><strong>});
</strong>
const prOidc = createOidc({ /* ... See below ... */ });

export async function getOidc(){
   const oidc = await prOidc;
   return oidc;
}
</code></pre>
{% endtab %}
{% endtabs %}

You might also want to enable some of the opt-in security features:

{% content-ref url="/broken/pages/9CHQX9V9dazRrd5n6par" %}
[Broken link](/broken/pages/9CHQX9V9dazRrd5n6par)
{% endcontent-ref %}
