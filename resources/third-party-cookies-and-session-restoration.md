---
icon: cookie
---

# Third‑party cookies and session restoration

{% hint style="success" %}
> **You’re safe by default** Even in the worst‑case scenario where your authorization server’s cookies are blocked by the browser, `oidc‑spa` automatically falls back to a near‑seamless full‑page redirect. **No configuration required.**
>
> That said, if you want the **best possible user experience** and the **strongest security guarantees** across all environments, it’s worth understanding what’s going on under the hood and configuring your domains and headers accordingly.
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

If the browser considers the IdP **third‑party** to your app, it often refuses to attach those cookies in an embedded context. Silent restoration then fails and the app must fall back to a full‑page redirect.

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

> **Checklist**
>
> * [ ] &#x20;Move the IdP to a subdomain of your app’s parent domain, or
> * [ ] &#x20;Configure a **custom domain** for your SaaS IdP (Auth0, Entra External ID/B2C, Clerk, etc.), and
> * [ ] &#x20;Update your OIDC `issuer` and endpoints to use that domain.

***

### How `oidc‑spa` restores sessions

`oidc‑spa` supports two session restoration strategies. You choose (or let the library auto‑choose) using `sessionRestorationMethod`.

```ts
bootstrapOidc({
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
* **Multiple OIDC clients in one page:** to avoid a redirect loop, the app may need to persist state between reloads (for example, tokens or a minimal session hint) which weakens the “no persistence” posture. Prefer the iframe strategy when you have multiple clients.

#### "auto" (default and recommended)

* `oidc‑spa` selects the best method at runtime. If your app and IdP share a parent domain and iframes are permitted, it uses "iframe". Otherwise it uses "full page redirect".

> **Migration note**
>
> The old `noIframe` option is **deprecated**. Use `sessionRestorationMethod: "full page redirect"` to get equivalent behavior.

***

### When iframes are blocked by policy

Some environments forbid iframes entirely. If your app is served with either of the following headers, even self‑iframes are blocked:

```
Content-Security-Policy: frame-ancestors 'none'
X-Frame-Options: DENY
```

If you can change the policy, allow self‑iframes:

```diff
- add_header X-Frame-Options "DENY";
- add_header Content-Security-Policy "frame-ancestors 'none'";
+ add_header Content-Security-Policy "frame-ancestors 'self'";
```

If you prefer to only allow iframes in the very specific case of a silent OIDC check, you can conditionally relax CSP based on query parameters at your reverse proxy:

```nginx
map $query_string $add_content_security_policy {
  "~*(?=.*\bstate=)(?=.*\bclient_id=)(?=.*\bresponse_type=)(?=.*\bredirect_uri=)" "frame-ancestors 'self'";
  default "frame-ancestors 'none'";
}
add_header Content-Security-Policy $add_content_security_policy;
```

`oidc‑spa` authenticates and encrypts cross‑window messages and validates origins so that allowing this narrowly scoped iframe remains consistent with your security model.

If changing headers is not possible, set `sessionRestorationMethod: "full page redirect"`.

***

### Local development

When your app runs on `localhost` and your IdP lives on a different domain, the browser treats the IdP as third‑party. Silent restoration with an iframe will fail. You have two options:

1. **Develop with real‑world conditions**: set `sessionRestorationMethod: "iframe"` and add an exception in your browser to allow third‑party cookies for your IdP domain. Remove the exception before shipping.
2. **Use a dev hostname under your parent domain**: e.g., map `dev.my-company.test` to `127.0.0.1` in `/etc/hosts` and serve your app there so it shares the parent domain with your IdP.

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

(The video is lowed down so you can perceive the difference)

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
