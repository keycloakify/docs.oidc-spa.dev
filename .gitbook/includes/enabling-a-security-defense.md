---
title: Enabling a security defense
---

{% tabs %}
{% tab title="Vite Plugin" %}
<pre class="language-typescript" data-title="vite.config.ts"><code class="lang-typescript">import { defineConfig } from "vite";
import { oidcSpa } from "oidc-spa/vite-plugin";

export default defineConfig({
    plugins: [
        // ...
        oidcSpa({
            // ...
<strong>            tokenSubstitution: {
</strong><strong>                enabled: true,
</strong><strong>                // Optional, see below
</strong><strong>                trustedThirdPartyResourceServers: [
</strong><strong>                    "s3.amazonaws.com", 
</strong><strong>                    "*.microsoft.com"
</strong><strong>                ]
</strong><strong>            }
</strong><strong>        })
</strong>    ]
});
</code></pre>
{% endtab %}

{% tab title="Manual" %}
<pre class="language-typescript" data-title="src/main.ts"><code class="lang-typescript">import { oidcEarlyInit } from "oidc-spa/entrypoint";
import { enableTokenSubstitution } from "oidc-spa/token-substitution";

const { shouldLoadApp } = oidcEarlyInit({
    // ...
<strong>    extraDefenseHook: () => {
</strong><strong>        enableTokenSubstitution({
</strong><strong>           // Optional, see below
</strong><strong>           trustedThirdPartyResourceServers: [
</strong><strong>              "s3.amazonaws.com", 
</strong><strong>              "*.microsoft.com"
</strong><strong>           ]
</strong><strong>        });
</strong><strong>    }
</strong>});

if (shouldLoadApp) {
    import("./main.lazy");
}
</code></pre>
{% endtab %}
{% endtabs %}
