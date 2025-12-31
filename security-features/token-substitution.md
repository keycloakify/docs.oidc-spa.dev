---
icon: cards-blank
---

# Token Substitution

{% hint style="danger" %}
Still under construction
{% endhint %}

## Uderstanding the Defence

The Token Substitution Defence is a mecanism that you can enable to protect against token exfiltration in case of successfull NPM supply chain attack or XSS.

When this mode is enabled the access token that you can manipulate at the application level are substituted by harmless token that can't be used to access resource server.

Concretely if you do:

```typescript
const acessToken = await oidc.getAccessToken();
```

You'll get a string that looks like this (<mark style="color:green;">\<header></mark><mark style="color:orange;">.</mark><mark style="color:yellow;">\<payload></mark><mark style="color:orange;">.</mark><mark style="color:purple;">\<signature></mark>):

<mark style="color:green;">eyJh...3NRIn0</mark><mark style="color:orange;">.</mark><mark style="color:yellow;">eyJleHA....QifQ</mark><mark style="color:orange;">.</mark><mark style="color:purple;">sig\_placeholder\_1640197\_AAA....AAA</mark>

The <mark style="color:green;">header</mark> and <mark style="color:yellow;">payload</mark> part are real and unlalterated, (so the token can still be decoded). The <mark style="color:purple;">signature</mark> part however will have been substituted by a placeholder, to it can't be validated.

Consequence: If an attacker exfiltrate this token they won't be able to use is to access resource server since the mock signatue will make any validation attempt fail. &#x20;

The placeholder signature will be replaced internally by the real signature at network APIs interceptor level set up by oidc-spa via the vite plugin or in the oidcEarly init.

fetch, XMLHttpRequext, WebSocket, Beacon, fetchLater are covered and the token can be anywhere, in the header, the body, the url, the interveptor will replace them transparently.\
\
Aditionally, the interceptor will enforce that only request to trusted resource server can go out. &#x20;

## VS DPoP

[DPoP](dpop.md) Posture: "Limits the security implication of a leaked token".

Token Substitution Posture: "Devence to prevent token from being leaked in the first place".

DPoP Nature: Protocol level RFC, open standard, relying on cryptography.

Token Substitution Nature: Adapter level strategy. Specific to oidc-spa, best effort: trying to treat the JavaScript runtime as hostile environement is very delicate, we can patch holes as they get discovered but not guarentee that there's no holes in the first place.

Overlap in security property: Limiting the severity of the harm that can be done by a successfull supply chain or XSS attack.

So DPoP is unquestionably a better defence than Token Substitution. If your all app is covered with DPoP you can safely ignore Token Exfiltration. But, the reality is that DPoP often can't be enabled everywhere:

* Not all Authorization server and Resource Server support DPoP yet.
* WebSocket is out of scope for DPoP.
* Some token exchange like AWS S3 STS or HashiCorp Vault, when you exchange an access token for an other kind of token, those calls are often requiring you to pass the access token in the payload body, making thos call fall outsideof the DPoP spec.

So if one of those three thing is true for you, you'll still benefit from enabling Token Substitution.

## Requirements; Can I enable it?

Unfortunately, the requirement for being able to enable token substitution are quite strict, not all apps will be able to.\
What you need:

* Having enabled [Browser Runtime Freeze,](browser-runtime-freeze.md) idealy with no exception, if the integrity of the environement can't be guarentied the token  substitution strategy can be easily circumvented.
* If your app consumes resource servers that are outside of your site like for example s3.amazon.com, you need to know at builtime (or at least syncronously at runtime) what are the urls of those servers. If we simply allow request to go out an attacker can just call their own server and receive the real token on the other end.&#x20;
* If you need to actually render the access toke to the user. You won't be able to implement a button "copy the access token" for example.

## Enabling the defence

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
</strong><strong>                resourceServersAllowedHostnames: [
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
</strong><strong>           // Optional:, see below
</strong><strong>           resourceServersAllowedHostnames: [
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

### resourceServersAllowedHostnames

Example: \["s3.amazonaws.com","\*.api.my-company.com"]

&#x20;    Note that any domains first party (same site) relative to where your app

&#x20;     is deployed will be automatically allowed.

&#x20;   &#x20;

&#x20;    So for example if your app is deployed under:

&#x20;     dashboard.my-company.com

&#x20;    \*Authed request to the following domains will automatically be allowed (examples):

&#x20;    \- minio.my-company.com

&#x20;    \- minio.dashboard.my-company.com

&#x20;    \- my-company.com

&#x20;   &#x20;

&#x20;    BUT there is an exception to the rule. If your app is deployed under free default domain

&#x20;    provided by known hosting platform like

&#x20;    \- xxx.vercel.com

&#x20;    \- xxx.netlify.com

&#x20;    \- xxx.github.com

&#x20;    \- xxx.pages.dev (firebase)

&#x20;    \- xxx.web.app (firebase)

&#x20;    \- ...

&#x20;   &#x20;

&#x20;    We we won't allow request to parent domain since those are multi tenant.

&#x20;   &#x20;

&#x20;    Also, all filtering will be disabled when the app is ran with the dev server, so under:

&#x20;    \- localhost

&#x20;    \- 127.0.0.1

&#x20;    \- \[::]&#x20;
