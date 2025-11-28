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

Silent SSO (iframe-based session restoration) only works if your app is allowed to open an iframe toward your IdP *and* if your app is allowed to be iframed by itself.  
If your CSP forbids either of these conditions, silent SSO will fail even if your IdP is first-party relative to your app.

If you do **not** control your server configuration, set:

```ts
sessionRestorationMethod: "full page redirect"
```

in your oidc-spa configuration.

---

## ❌ CSP rules that break silent SSO

Silent SSO breaks under two categories:

---

### **1) Your app cannot iframe the IdP**

This happens when:

- `X-Frame-Options: DENY`
- `Content-Security-Policy: frame-src 'none'`
- `Content-Security-Policy: frame-src 'self'`
- `Content-Security-Policy: frame-src 'self' https://not-my-idp.com`

If the IdP domain is missing from `frame-src`, the iframe cannot load → silent SSO cannot run.

---

### **2) Your app cannot be iframed by itself**

Silent SSO needs to temporarily load your app inside an iframe (when the IdP redirects back with the authorization response).

If you block this:

- `Content-Security-Policy: frame-ancestors 'none'`

…then the IdP cannot redirect to your app inside the iframe → silent SSO fails.

---

## ✅ How to fix it

To restore silent SSO:

1. **Remove** any `X-Frame-Options` header (deprecated).
2. **Allow** the IdP domain in `frame-src`.
3. **Allow** your app to frame itself using `frame-ancestors 'self'`.

Example:

```
Content-Security-Policy:
  frame-src https://auth.my-domain.com;
  frame-ancestors 'self';
  ...other CSP directives...
```

### **Tip**  
Instead of hardcoding the IdP domain, allow *sibling subdomains* of your app’s domain.  
This stays aligned with same-site cookie rules and avoids config drift between environments.

The Nginx configuration below demonstrates this pattern.

---

## ⭐ Canonical Nginx configuration

This is a strict, production-grade CSP that:

- supports Vite hashed assets,  
- forbids all external scripts (strict-dynamic),  
- disables all workers (service + web),  
- **still allows iframe-based silent SSO**,  
- and is suitable for SPA deployments using the image `nginxinc/nginx-unprivileged`.

Adapt if you rely on CDN assets or service workers (via nonce or SHA).

```nginx
# ============================================================
# Dynamic base domain extraction (per request)
# Example: datalab.sspcloud.fr -> sspcloud.fr
# ============================================================
map $host $base_domain {
    ~^(?<sub>.+)\.(?<domain>[^.]+\.[^.]+)$  $domain;
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
            "frame-src https://*.$base_domain https://$base_domain; "
            "frame-ancestors 'self'; "
            "object-src 'none'; "
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
```

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
