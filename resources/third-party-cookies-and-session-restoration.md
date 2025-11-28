---
icon: cookie
---

# Third‑party cookies and session restoration

{% hint style="success" %}
> **You’re safe by default** Even in the worst‑case scenario where your authorization server’s cookies are blocked by the browser, `oidc‑spa` automatically falls back to a near‑seamless full‑page redirect. **No configuration required.**
>
> That said, if you want the **best possible user experience**, it’s worth understanding what’s going on under the hood and configuring your domains and headers accordingly.
{% endhint %}

This page explains why modern browsers often refuse to send cookies in third‑party contexts, how that impacts silent session restoration in frontend centric auth model, and how to configure your domain and security headers so that `oidc‑spa` can deliver a seamless UX.

> TL;DR
>
> 1. Align your application and authorization endpoint under a common parent domain so the browser treats your IdP as first‑party to your app.&#x20;
> 2. Prefer iframe‑based restoration when possible.&#x20;
> 3. If your CSP forbids iframes or the IdP must live on a foreign domain, use full‑page redirects.

***

### Why third‑party cookies matter here

Traditional web apps keep a session on your backend. Your browser sends the backend’s own cookies on every request, so restoring the user session is trivial.

With `oidc‑spa`, your frontend talks directly to the authorization server. When a user revisits your app, `oidc‑spa` first tries to learn whether the user still has a valid session **at the IdP** without prompting for credentials again. It does so by contacting the authorization endpoint silently. If the browser **sends the IdP’s cookies** in that context, the IdP can attest that the user is still signed in and return the data needed to rebuild local identity.

If the browser considers the IdP **third‑party** to your app, it often refuses to attach those cookies in an embedded context. oidc-spa has to use full‑page redirect in those configurations.

***

### Make your IdP first‑party: share a parent domain

The key is to host your application and your authorization endpoint under the **same registrable (parent) domain**.

#### ✅ Examples where the IdP is _not_ third‑party

* App: `www.my-company.com`, `dashboard.my-company.com`, or `my-company.com/dashboard`
* Authorization endpoint: `https://auth.my-company.com/realms/oidc-spa/protocol/openid-connect/auth`
* **Parent domain:** `my-company.com`

#### ❌ Examples where the IdP _is_ third‑party

* App: `my-company.com`
* Authorization endpoints on unrelated domains:
  * `https://auth.my-keycloak.com/realms/oidc-spa/protocol/openid-connect/auth` _(configurable; you choose where to host)_
  * `https://login.microsoftonline.com/<tenant>/oauth2/v2.0/authorize` _(configurable via External ID / B2C custom domain)_
  * `https://<tenant>.us.auth0.com/authorize` _(configurable via Auth0 Custom Domains)_
  * `https://accounts.google.com/o/oauth2/v2/auth` _(not configurable)_

***

### How `oidc‑spa` restores sessions

`oidc‑spa` supports two session restoration strategies. You choose (or let the library auto‑choose) using `sessionRestorationMethod`.

```ts
bootstrapOidc({ // or createOidc({
  // "auto" (default) | "iframe" | "full page redirect"
  sessionRestorationMethod: "auto"
});
```

#### "iframe" (silent, seamless)

* The app opens an **invisible iframe** to the authorization endpoint with parameters that request a silent check.
* If the IdP’s cookies are present, the IdP returns enough data for the app to rebuild identity without leaving the page.
* **Best UX.** Requires that the browser can send IdP cookies in the iframe and that your app’s security headers allow same‑origin iframes.

#### "full page redirect" (silent but navigational)

* The app performs a quick top‑level redirect to the authorization endpoint, which always carries IdP cookies.
* The redirect returns immediately to your app with the information needed to rebuild identity.
* **Works everywhere** but is a about 30% slower and the url flashes auth response info brievly.
* **Multiple OIDC clients in one page:** to avoid a redirect loop, the app may need to persist state between reloads (for example, tokens or a minimal session hint) which weakens the “no persistence” posture.&#x20;

#### "auto" (default and recommended)

* `oidc‑spa` selects the best method at runtime. If your app and IdP share a parent domain and iframes are permitted, it uses "iframe". Otherwise it uses "full page redirect".

> **Migration note**
>
> The old `noIframe` option is **deprecated**. Use `sessionRestorationMethod: "full page redirect"` to get equivalent behavior.

***

### When iframes are blocked by your CSP

In this section we will see how you can configure your CSP header as strictly as possible while enabling iframe session restoration to work. If you don't have control over your server configuration just use  set `sessionRestorationMethod: "full page redirect"` in your oidc-spa config.

Tere are two type of policy that will lead to the silent SSO to fail even if your IdP is first party relative to your app:

1\) You explicitely forbiden the popening of an ifram toward your IdP domain, this is the case if the HTML of your app is served with the following HTTP header response:&#x20;

X-Frame-Options -> DENY

Content-Security-Policy   frame-src: 'none' ... or frame-src: 'self' ... or frame-src: 'self' https://not-my-idp.com

2\) You explicitely forbien your app to be iframed

Content-Security-Policy   frame-ancestors: 'none'&#x20;

How to fix it:

Here are the more restrictice CSP you can apply while still having silent SSO session restoration working

First remove the X-Frame-Options header if you have it, it's deprecated in favor of Content-Security-Policy.

Then you want to have something like:

Content-Security-Policy -> frame-src: https://auth.my-domain.com; frame-ancestors 'self' ... other CSP;

in frame-src you should allow the domain of the authorization endpoint of your IdP, the best approach is not to hardcode it in your config since it will be a pain to maintain but to allow any sibling domain of where your app is hosted, this matches the condition for cookie to be sucessfully set. See after for how to configure it with Ngnix.\
Frame-ancestors 'self' is also mandatory. You won't be iframing your app directly but the IdP will issue a redirect to your app with the code attached as query param. So you want your app to allow to be iframed by itself.  <br>

Canonical Ngnix configuration (For a Docker image running nginxinc/nginx-unprivileged, [example](https://github.com/InseeFrLab/onyxia/blob/9aae4005712120d7fc8830589350a1ecd742b3b9/web/Dockerfile)): &#x20;

This is a Typical ngnix config for an SPA that is portable and apply as strict at can be content security policy that will still enable the silent SSO session restoration via iframe to work,

In this CSP, with worker-src, child-src, and script-src, we forbid loading all scripts that ar not hosted by the app and forbid all workers. This is extreem, adapt if you are using CDN or are using web/service worker, use nonce or hash sha if you have legitimate inline script.<br>

<pre class="language-nginx" data-title="ngnix.conf"><code class="lang-nginx"># ============================================================
# Dynamic base domain extraction (per request)
# Extracts the last two labels from $host
# Example: datalab.sspcloud.fr -> sspcloud.fr
# ============================================================
map $host $base_domain {
    ~^(?&#x3C;sub>.+)\.(?&#x3C;domain>[^.]+\.[^.]+)$  $domain;
    default                                 $host;
}

server {
    listen 8080;

    # -------------------------
    # Gzip
    # -------------------------
    gzip on;
    gzip_vary on;
    gzip_min_length 1024;
    gzip_proxied expired no-cache no-store private auth;
    gzip_types
        text/plain text/css text/xml text/javascript
        application/javascript application/x-javascript application/xml;
    gzip_disable "MSIE [1-6]\.";

    # -------------------------
    # Root and SPA routing
    # -------------------------
    root /usr/share/nginx/html;
    index index.html;

    # Serve SPA; CSP applied in the HTML location block
    location / {
        try_files $uri $uri/ /index.html;
    }

    # -------------------------
    # Vite hashed assets (cache 1 year)
    # -------------------------
    location ^~ /assets/ {
        try_files $uri =404;
        expires 1y;
        access_log off;
        add_header Cache-Control "public" always;
    }

    # -------------------------
    # HTML (never cached) + CSP
    # -------------------------
    location ~* \.html$ {
        try_files $uri =404;
        expires -1;
        add_header Content-Security-Policy
<strong>            "frame-src https://*.$base_domain https://$base_domain; "
</strong><strong>            "frame-ancestors 'self'; "
</strong>            "object-src 'none'; "
            "worker-src 'none'; "
            "child-src 'none'; "
            "script-src 'self' 'strict-dynamic';"
            always;
    }

    # -------------------------
    # JSON / TXT (never cached)
    # -------------------------
    location ~* \.(json|txt)$ {
        try_files $uri =404;
        expires -1;
    }

    # -------------------------
    # Any other file with an extension (cache 1 day)
    # -------------------------
    location ~ ^.+\..+$ {
        try_files $uri =404;
        expires 1d;
        access_log off;
        add_header Cache-Control "public" always;
    }
}
</code></pre>

***

### Local development

When your app runs on `localhost` and your IdP lives on a different domain, wichis almost always the case unless you run a keycloak locally.&#x20;

The browser treats the IdP as third‑party so oidc-spa will fallback to full page redirect. To run your app in devloppement like you would in prod you need to:

1. Set `sessionRestorationMethod: "iframe"` explicitely to force oidc-spa to use iframe.
2. Allow third party cookies in localhost:

<figure><img src="../.gitbook/assets/image (4).png" alt="" width="348"><figcaption></figcaption></figure>



***

### SaaS IdPs and custom domains

Most managed IdPs let you put their endpoints behind your domain. This is crucial to avoid third‑party treatment.

* **Auth0**: supports **Custom Domains** for the authorization and token endpoints.
* **Microsoft Entra External ID / Azure AD B2C**: supports **Custom URL domains** for user flows and policies, which cover the authorization endpoint.
* **Clerk**: supports **custom and satellite domains** and proxying its Frontend API so your app interacts under your domain.
* **Google**: the authorization endpoint is always `accounts.google.com`. You cannot host it under your domain. If you need Google login, consider fronting multiple IdPs with an aggregator like Keycloak, Auth0 or Clerk that itself lives under your domain.

***

### UX comparison

A short video that shows the UX difference between iframe‑based restoration and a full‑page redirect:

(This video was recorded a while ago, performance a **much** better now)

{% embed url="https://www.youtube.com/watch?v=55sZ7XSWh4Q" %}

***

### Decision guide

1. Can your app and IdP share a parent domain?
   * **Yes** → Use `sessionRestorationMethod: "auto"` (will pick "iframe").
   * **No** → Use `"full page redirect"`.
2. Do your security headers allow self‑iframes?
   * **Yes** → You are set.
   * **No** → Either relax to `frame-ancestors 'self'` or stick to `"full page redirect"`.
3. Do you host multiple OIDC clients in one page?
   * **Yes** → Strongly prefer iframe restoration to avoid redirect loops and persistence.

***

### API reference

```ts
/**
 * Controls how session restoration is handled.
 * "auto" picks the best strategy at runtime.
 */
sessionRestorationMethod?: "iframe" | "full page redirect" | "auto";

/**
 * @deprecated Use `sessionRestorationMethod: "full page redirect"` instead.
 */
noIframe?: boolean;
```
