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

If you are using the mock with the vanilla API (not React)

```diff
 import { createOidc } from "oidc-spa";
 import { createMockOidc } from "oidc-spa/mock";
 import { z } from "zod";

 const decodedIdTokenSchema = z.object({
     sub: z.string(),
     preferred_username: z.string()
 });

 const publicUrl= import.meta.env.BASE_URL;

 const oidc = !import.meta.env.VITE_OIDC_ISSUER
-    ? await createMockOidc({
+    ? await createMockOidc({
           isUserInitiallyLoggedIn: false,
           publicUrl,
           mockedTokens: {
               decodedIdToken: {
                   sub: "123",
                   preferred_username: "john doe"
               } satisfies z.infer<typeof decodedIdTokenSchema>
           }
       })
     : await createOidc({
           issuerUri: import.meta.env.VITE_OIDC_ISSUER,
           clientId: import.meta.env.VITE_OIDC_CLIENT_ID,
           publicUrl
       });
```

[^1]: Yes .htm and not .html.  \
    Some web server such as serve will rewrite /silent-sso.html to /silent-sso and serve the index.html of your distribution.  \
    Using the .htm extention prevend any potential issue.
