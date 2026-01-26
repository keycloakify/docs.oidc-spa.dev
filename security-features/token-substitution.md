---
icon: cards-blank
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/oygeayjvIPxroUcp3jt4/security-features/token-substitution
---

# Token Substitution

### Enabling the defence

{% tabs %}
{% tab title="Vite Plugin" %}
<pre class="language-typescript" data-title="vite.config.ts"><code class="lang-typescript">import { defineConfig } from "vite";
import { oidcSpa } from "oidc-spa/vite-plugin";

export default defineConfig({
    plugins: [
        // ...
        oidcSpa({
            // ...
            browserRuntimeFreeze: { enabled: true, /*exclude: [...]*/ }, //Recommended
<strong>            tokenSubstitution: {
</strong><strong>                enabled: true,
</strong><strong>                // Optional, see below
</strong><strong>                trustedExternalResourceServers: [
</strong><strong>                    "*.{{location.hostname}}",
</strong><strong>                    "s3.amazonaws.com"
</strong><strong>                ]
</strong><strong>            }
</strong><strong>        })
</strong>    ]
});
</code></pre>
{% endtab %}

{% tab title="Manual" %}
<pre class="language-typescript" data-title="src/main.ts"><code class="lang-typescript">import { oidcEarlyInit } from "oidc-spa/entrypoint";
import { browserRuntimeFreeze } from 'oidc-spa/browser-runtime-freeze';
<strong>import { tokenSubstitution } from 'oidc-spa/token-substitution';
</strong>
const { shouldLoadApp } = oidcEarlyInit({
    BASE_URL: "/",
    securityDefenses: {
        ...browserRuntimeFreeze(/*{ exclude: [...] }*/), // Recommended
<strong>        ...tokenSubstitution({
</strong><strong>           // Optional, see below
</strong><strong>           trustedExternalResourceServers: [
</strong><strong>               `*.${location.hostname}`,
</strong><strong>               "s3.amazonaws.com"
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

[Assuming your Authorization Server issues, JWT access token](#user-content-fn-1)[^1], you’ll get a string shaped like:

`<real header>`<mark style="color:orange;">`.`</mark>`<real payload>`<mark style="color:orange;">`.`</mark><mark style="color:yellow;">`<placeholder signature>`</mark>

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

DPoP is generally the stronger defence but in practice **but**:

* not all authorisation servers and resource servers support DPoP yet
* [WebSocket is out of scope for DPoP](../integration-guides/backend-token-validation/websocket.md)
* some token exchanges require the access token in the request body (often outside DPoP’s coverage), e.g. AWS STS or [Vault-style exchanges](../features/talking-to-multiple-apis-with-different-access-tokens.md#using-multiple-clients-in-oidc-spa)

If any of these apply, enabling Token Exfiltration still improve your security posture significantly.

### Requirements (can I enable it?)

The requirements are strict. Not every app can enable it.

You need:

* To enable [Browser Runtime Freeze](browser-runtime-freeze.md).\
  If runtime integrity can’t be guaranteed, this defence can be bypassed.\
  That being said the defence remains effective even if you had to exclude fetch and XMLHttpRequest. (`browserRuntimeFreeze.exclude = ["fetch", "XMLHttpRequest"]`)
* If you call resource servers outside your site (example: `s3.amazonaws.com`), you must know their hostnames at build time (or synchronously at runtime).
* You must not need to display the raw access token to the user.\
  Example: no “copy access token” button.

### trustedExternalResourceServers

Use this when your app needs to call resource servers **outside** your host.

By default, oidc-spa only allows authenticated requests to your own origin (`location.hostname`) so you can call  `fetchWithAuth("/api/todos")` with configuring anything.

If your code tries to send an authenticated request to another host, it gets blocked.

#### What to put in the list

Each entry is a **hostname pattern** (not a URL).

Supported shapes:

* Exact host: `"s3.amazonaws.com"`
* Any subdomain: `"*.my-company.com"`

You don’t need to include `location.hostname`. It’s always allowed.

#### Example

If you want to allow any subdomain of your base domain (same site) plus S3 STS:

```typescript
trustedExternalResourceServers: [
  "*.{{location.hostname.split('.').slice(-2).join('.')}}",
  "s3.amazonaws.com"
]
```

At runtime, if your app is hosted at `dashboard.my-company.com`:

```typescript
trustedExternalResourceServers: [
  "*.my-company.com",
  "s3.amazonaws.com"
]
```

oidc-spa will allow authenticated requests to hosts matching these patterns. Any other host is treated as untrusted.

{% hint style="info" %}
Host filtering is disabled in dev server environments:

* `localhost`
* `127.0.0.1`
* `[::]`
{% endhint %}

[^1]: If your IdP issues opaque tokens, you'll just receive a placeholder string.
