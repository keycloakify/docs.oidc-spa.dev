---
icon: cards-blank
---

# Token Substitution

{% hint style="danger" %}
Still under construction
{% endhint %}

### Understanding the defence

Token Substitution is an optional defence against token exfiltration during:

* a successful NPM supply-chain attack
* an XSS compromise

When enabled, any access token your app can read is replaced with a harmless “substituted” token.\
The substituted token **cannot** be used to call a resource server.

Example:

```typescript
const accessToken = await oidc.getAccessToken();
```

You’ll get a JWT-shaped string:

`<real header>.<real payload>.<placeholder signature>`

The header and payload are real and unaltered.\
The signature is replaced with a placeholder, so validation fails.

#### What this blocks

If an attacker exfiltrates the substituted token, they can’t use it against your resource servers.\
Any validation attempt fails because the signature is fake.

#### How requests still work

oidc-spa restores the real signature **right before the request leaves the browser**.\
This happens inside hardened interceptors installed during early init (via the Vite plugin or `oidcEarlyInit`).

Covered APIs:

* `fetch`
* `XMLHttpRequest`
* `WebSocket`
* `navigator.sendBeacon`
* `fetchLater`

The token can be in headers, body, or URL.\
The interceptor replaces it transparently.

It also blocks authenticated requests to untrusted hosts.

### Compared to DPoP

Posture:

* [DPoP](dpop.md): limits the impact of a leaked token.
* Token Substitution: Attempts to prevent tokens from being leaked in the first place.

Nature:

* DPoP is a protocol-level RFC. It’s standardised and crypto-based.
* Token Substitution is an adapter-level technique. It’s oidc-spa specific.\
  It’s practical hardening, not a cryptographic guarantee.\
  It can’t be proven “bulletproof”. New bypasses may be found.

Overlap:

* Both reduce the damage from a successful supply-chain or XSS attack.

DPoP is generally the stronger defence but in practice, DPoP can't al

In practice, DPoP is not always possible:

* not all authorisation servers and resource servers support DPoP yet
* WebSocket is out of scope for DPoP
* some token exchanges require the access token in the request body (often outside DPoP’s coverage), e.g. AWS STS or Vault-style exchanges

If any of these apply, Token Substitution still helps.

### Requirements (can I enable it?)

The requirements are strict. Not every app can enable it.

You need:

* [Browser Runtime Freeze](browser-runtime-freeze.md), ideally with no exceptions.\
  If runtime integrity can’t be guaranteed, this defence can be bypassed.
* If you call resource servers outside your site (example: `s3.amazonaws.com`), you must know their hostnames at build time (or synchronously at runtime).\
  Otherwise an attacker could send a request to their own host and recover the real token.
* You must not need to display the raw access token to the user.\
  Example: no “copy access token” button.

### Enabling the defence

{% tabs %}
{% tab title="Vite Plugin" %}
<pre class="language-typescript" data-title="vite.config.ts"><code class="lang-typescript">import { defineConfig } from "vite";
import { oidcSpa } from "oidc-spa/vite-plugin";

export default defineConfig({
    plugins: [
        // ...
        oidcSpa({
<strong>            tokenSubstitution: {
</strong><strong>                enabled: true,
</strong><strong>                // Optional, see below
</strong><strong>                trustedThirdPartyResourceServers: [
</strong><strong>                    "s3.amazonaws.com", 
</strong><strong>                    "*.api.my-company.com"
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
    extraDefenseHook: () => {
<strong>        enableTokenSubstitution({
</strong><strong>           // Optional, see below
</strong><strong>           trustedThirdPartyResourceServers: [
</strong><strong>              "s3.amazonaws.com", 
</strong><strong>              "*.api.my-company.com"
</strong><strong>           ]
</strong><strong>        })
</strong>    }
});

if (shouldLoadApp) {
    import("./main.lazy");
}
</code></pre>
{% endtab %}
{% endtabs %}

### trustedThirdPartyResourceServers

Use this when your app needs to call third-party resource servers (outside your site).

Example:

```ts
["s3.amazonaws.com", "*.api.my-company.com"]
```

#### What’s allowed by default

Same-site (first-party) hosts are allowed automatically.

Example: if your app is deployed at `dashboard.my-company.com`, these are allowed:

* `minio.my-company.com`
* `minio.dashboard.my-company.com`
* `my-company.com`

{% hint style="warning" %}
If your app is deployed under a free multi-tenant domain, parent domains are **not** automatically allowed.

Examples:

* `xxx.vercel.app`
* `xxx.netlify.app`
* `xxx.github.io`
* `xxx.pages.dev`
* `xxx.web.app`
{% endhint %}

{% hint style="info" %}
Host filtering is disabled in dev server environments:

* `localhost`
* `127.0.0.1`
* `[::]`
{% endhint %}
