---
icon: crop-simple
---

# iframe related issues

By default, applications using **oidc-spa** will create an iframe pointing to themselves in order to quickly restore the user’s session across reloads and navigations.

However, iframes have a bad reputation. They are sometimes considered an attack vector, and certain system engineers may prefer to forbid their usage entirely rather than applying more nuanced policies.

If your application is served with response headers such as:

`Content-Security-Policy: frame-ancestors "none"`

or

`X-Frame-Options: DENY`

...then your operations team has completely blocked iframe usage, even your SPA is not allowed to iframe itself.

In this scenario, you have two options :

### Option 1: Enable the `noIframe` mode of oidc-spa

**oidc-spa** provides an option to disable iframe usage:

{% code title="src/oidc.ts" %}
```typescript
createReactOidc({
  // ...
  noIframe: true
});
```
{% endcode %}

Note: this will slightly increase the initialization time of your application. Everything will still work as expected, but you won't have the fastest possible startup.

### Option 2: Adjust your security policy to allow iframe usage in this context

If possible, request a change in the security policy from your ops team.\
Instead of strict policies like:

* `Content-Security-Policy: frame-ancestors 'none'`
* `X-Frame-Options: DENY`

...you can use a more permissive directive like:

* `Content-Security-Policy: frame-ancestors 'self'`

{% code title="nginx.conf" %}
```diff
-add_header X-Frame-Options "DENY";
-add_header Content-Security-Policy "frame-ancestors 'none'";
+add_header Content-Security-Policy "frame-ancestors 'self'";
```
{% endcode %}

If you'd like to allow iframe usage only in the specific case where the app is iframing itself with typical OIDC query parameters, you can use conditional logic in your reverse proxy:

{% code title="nginx.conf" overflow="wrap" %}
```nginx
map $query_string $add_content_security_policy {
  "~*(?=.*\bstate=)(?=.*\bclient_id=)(?=.*\bresponse_type=)(?=.*\bredirect_uri=)" "frame-ancestors 'self'";
  default "frame-ancestors 'none'";
}
add_header Content-Security-Policy $add_content_security_policy;
```
{% endcode %}

This configuration allows iframes only when the request query string includes the expected OIDC parameters, typically when the app is restoring a session by iframing itself.
