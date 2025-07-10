# iframe related issues

By default, your application that implements oidc-spa will create an iframe to itself to quickly restore use's session across reload and navigations. &#x20;

However iframe have a bad rep, they are known are being an attack vector in some scenario and some system engenneer would rather forbid their usage alltogether than having more nuanced policies. &#x20;

If, when you request your app, you can see in the response headers:

`Content-Security-Policy: frame-ancestors "none"`

or&#x20;

`X-Frame-Options: DENY`

Your ops team has completely provided the usage on iframe, your SPA is not even allowed to iframe itself.  \
\
In this senario you have two option:

### Enabling the "noIframe" mode of oidc-spa

There is an option to tell oidc-spa to do without iframe: &#x20;

{% code title="src/oidc.ts" %}
```typescript

createReactOidc({
   //...
   noIframe: true
})
```
{% endcode %}

Note however that your app initializatino time will take a little hit. Everything will work but you won't get the best acheivable initialization time. &#x20;

### Change your security policy to allow the usage of iframe in this contex

If you can, open a ticket to your ops team to soften the security policy regarding iframe.  \
Instead of using Content-Security-Policy: frame-ancestors 'none' or X-Frame-Options: DENY you could use Content-Security-Policy: frame-ancestors "self".&#x20;

{% code title="ngnix.config" %}
```diff
-add_header X-Frame-Options "DENY"
-add_header Content-Security-Policy "frame-ancestors 'none'";
+add_header Content-Security-Policy "frame-ancestors 'self'";
```
{% endcode %}

&#x20;\
If you want to only allow the usage of iframe in this very specific context you can create a rule in your reverse proxy like&#x20;

{% code title="" overflow="wrap" %}
```nginx
map $query_string $add_content_security_policy {
    "~*(?=.*\bstate=)(?=.*\bclient_id=)(?=.*\bresponse_type=)(?=.*\bredirect_uri=)" "frame-ancestors 'self'";
    default "frame-ancestors 'none'";
}
add_header Content-Security-Policy $add_content_security_policy;
```
{% endcode %}

So that iframe are only allowed when your app iframes itself and there is the usual oidc query params. \
