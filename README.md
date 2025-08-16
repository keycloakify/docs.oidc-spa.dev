---
icon: up
---

# v6 -> v7

The main change brougt by this major version is a better management token renewal.  \
The tokens are now renewer much less agressively.  \
Before we oidc-spa would make sure that there's a valid access\_token at all time in caches. This was problematic for config with short access token ttl (like 20s) oidc-spa will constently hit the OIDC server, even when the tab is not focussed, over and over, in the background.  \
Now we get a new token only when and if we need one.  \
We only automatically refresh the tokens when the refresh token is about to expire so that we keep the session active on the oidc server.  \
The inactivity of the user is tracked locally by oidc-spa, the server can't relyably tell if the user is inactive just because he hasn't sent a request in a while, he could just be taking some time to fill a form. &#x20;

\
To make this change possible one big API change had to be made: The getTokens() method is now async. &#x20;

```diff
-const tokens = oidc.getTokens();
+const tokens = await oidc.getTokens();
```

And since we had to make this change we took the opportunity to do a few other wellcome API adjustment.  \
\
The tokens are no longer accessible in your react components.  You can no longer write: &#x20;

```diff
-const { tokens } = useOidc({ assert: "user logged in" });
-console.log(tokens!.accessToken);
```

There is no alternative for this since the tokens really do not bellong in the UI rendering world.  \
In the UI, what you might need is the information about the user, those informations are available in the decodedIdToken, which is still available via the react hook: &#x20;

```tsx
function HeaderButtonLoggedIn(){
    const { decodedIdToken } = useOidc({ assert: "user logged in" });
    
    return <div>Hello {decodedIdToken.name}</div>;
}
```

But the access token, no, it has nothing to do in a React component, it should be opaque to your SPA code and only be used as an authorization Bearer, example of valid usage:

{% code title="Custom fetch function that auto add the Authorization header." %}
```typescript
export const fetchWithAuth: typeof fetch = async (input, init) => {
    const oidc = await getOidc();

    if (oidc.isUserLoggedIn) {
        const { accessToken } = await oidc.getTokens();

        (init ??= {}).headers = {
            ...init.headers,
            Authorization: `Bearer ${accessToken}`
        };
    }

    return fetch(input, init);
};
```
{% endcode %}

If you really need to have the access token in your react component there is a custom hook example here: [https://github.com/keycloakify/oidc-spa/blob/d1eede4e0cec6b475a3dbc2b4677b6a02326c05d/examples/tanstack-router-file-based/src/routes/protected.tsx#L120-L174](https://github.com/keycloakify/oidc-spa/blob/d1eede4e0cec6b475a3dbc2b4677b6a02326c05d/examples/tanstack-router-file-based/src/routes/protected.tsx#L120-L174)



Other minor API changes: &#x20;

\_\_unsafe\_ssoSessionIdleSeconds was renamed idleSessionLifetimeInSeconds

```diff
 createReactOidc({
-    __unsafe_ssoSessionIdleSeconds: 60 * 10,
+    idleSessionLifetimeInSeconds: 60 * 10,

-    transformUrlBeforeRedirect: url => `${url}&ui_locale=fr`,
+    transformUrlBeforeRedirect: ({ authorizationUrl, isSilent }) => {
+        if(isSilent){
+           return authorizationUrl;
+        }
+        return `${url}&ui_locale=fr`;
+    }
 })
```

\
