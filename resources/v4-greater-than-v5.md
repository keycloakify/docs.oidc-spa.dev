# ⬆️ v4 -> v5

Very little change in the API. &#x20;

* The public/silent-sso.html -> to public/silent-sso.htm ([without the l](#user-content-fn-1)[^1])&#x20;
* Use `getOidc()` instead or `prOidc` (React API)

{% code title="src/oidc.ts" %}
```diff
 import { createReactOidc } from "oidc-spa/react";
 
 export const {
     OidcProvider,
     useOidc,
-    prOidc,
+    getOidc
 } = createReactOidc({ });

-prOidc.then(oidc => { /* ... */ });
+getOidc().then(oidc => { /* ... */ });
```
{% endcode %}

[^1]: Yes .htm and not .html.  \
    Some web server such as serve will rewrite /silent-sso.html to /silent-sso and serve the index.html of your distribution.  \
    Using the .htm extention prevend any potential issue.
