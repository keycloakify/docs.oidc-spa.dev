---
icon: user-police
---

# CSP Configuration

In this page we will see how to relax your Content-Security-Policy just enough so session restoration using iframe is possible. &#x20;

## CSP rules that break session restoration via iframe

Silent session restoration via iframe is **optional**, oidc-spa can restore user session just fine using full page redirect.

_If that is so, why does oidc-spa even attemt to use iframe?_

* For performance: iframe based session resoration is a little faster than full page redirect (not much but still noticable).
* For security **if and only if** [your app talks to more than one resource server](../talking-to-multiple-apis-with-different-access-tokens.md) (which is **not** the case in most apps), because "multiple oidc-client" + "no iframe SSO" = "oidc-spa needs to persist tokens in session storage". &#x20;

***

### **1) Your app cannot iframe the IdP**

This happens when:

* `X-Frame-Options: DENY`
* `Content-Security-Policy: frame-src 'none'`
* `Content-Security-Policy: frame-src 'self'`
* `Content-Security-Policy: frame-src 'self' https://not-my-idp.com`

If the IdP domain is missing from `frame-src`, the iframe cannot load → silent SSO cannot run.

***

### **2) Your app cannot be iframed by itself**

Silent SSO needs to temporarily load your app inside an iframe (when the IdP redirects back with the authorization response).

If you block this:

* `Content-Security-Policy: frame-ancestors 'none'`

…then the IdP cannot redirect to your app inside the iframe → silent SSO fails.

***

## How to fix it

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

Tip: Instead of hardcoding the IdP domain, allow _sibling subdomains_ of your app’s domain.\
This stays aligned with same-site cookie rules and avoids config drift between environments.

The Nginx configuration below demonstrates this pattern.

***

## Canonical Nginx configuration

This is a portable, production-grade Ngnix config for an SPA with "as strict as can be" CSPs that still enable oidc-spa to use iframe for performing session restoration. &#x20;

Of course you can relax thoses CSP to meet the specific need of your app. This is just an example of what you would use if you use no service/web workers and don't have any inline script. &#x20;

<pre class="language-nginx" data-title="ngnix.conf"><code class="lang-nginx"># Assuming nginxinc/nginx-unprivileged

# ============================================================
# Dynamic base domain extraction (per request)
# Example: dashboard.my-company.com -> my-company.com
# ============================================================
<strong>map $host $base_domain {
</strong><strong>    ~^(?&#x3C;sub>.+)\.(?&#x3C;domain>[^.]+\.[^.]+)$  $domain;
</strong><strong>    default                                 $host;
</strong><strong>}
</strong>
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
<strong>        add_header Content-Security-Policy "
</strong><strong>            frame-src https://*.$base_domain;
</strong><strong>            frame-ancestors 'self';
</strong><strong>            object-src 'none';
</strong><strong>            worker-src 'none';
</strong><strong>            script-src 'self' 'strict-dynamic';
</strong><strong>        " always;
</strong>    }

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
