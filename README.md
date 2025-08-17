---
icon: up
---

# v6 -> v7

The main change in this major release is **improved token renewal management**.\
Tokens are now refreshed far less aggressively:

* **Before**: `oidc-spa` would always try to keep a valid `access_token` in cache.\
  With short-lived tokens (e.g. 20s TTL), this caused constant requests to the OIDC server, even when the tab was inactive.
* **Now**: a new token is only requested **when needed**.
* Automatic refresh happens only when the **refresh token is close to expiring**, ensuring the session remains active on the OIDC server.
* User inactivity is tracked **locally** by `oidc-spa`. The server cannot reliably infer inactivity just because no requests were sent (the user might just be filling a form).

***

## Breaking API Change: `getTokens()` is now async

To support this new behavior, the `getTokens()` method has become asynchronous:

```diff
-const tokens = oidc.getTokens();
+const tokens = await oidc.getTokens();
```

***

## Displaying a warning Overlay Before Auto Logout in React.

The React API have been simplified: &#x20;

```diff
 import { useOidc } from "../oidc";
 
 export function AutoLogoutWarningOverlay() {
 
-    const { isUserLoggedIn, subscribeToAutoLogoutCountdown } = useOidc();
-    const [secondsLeft, setSecondsLeft] = useState<number | undefined>(undefined);
-    useEffect(
-        () => {
-            if (!isUserLoggedIn) {
-                return;
-            }
-
-            const { unsubscribeFromAutoLogoutCountdown } = subscribeToAutoLogoutCountdown(
-                ({ secondsLeft }) =>
-                    setSecondsLeft(
-                        secondsLeft === undefined || secondsLeft > 60 ? undefined : secondsLeft
-                    )
-            );
-
-            return () => {
-                unsubscribeFromAutoLogoutCountdown();
-            };
-        },
-        [isUserLoggedIn, subscribeToAutoLogoutCountdown]
-    );
+    const { useAutoLogoutWarningCountdown } = useOidc();
+    const { secondsLeft } = useAutoLogoutWarningCountdown({ warningDurationSeconds: 60 });
 
 
     if (secondsLeft === undefined) {
         return null;
     }
 
     return (
         <div
             // Full screen overlay, blurred background
             style={{
                 position: "fixed",
                 top: 0,
                 left: 0,
                 right: 0,
                 bottom: 0
             }}
         >
             <div>
                 <p>Are you still there?</p>
                 <p>You will be logged out in {secondsLeft}</p>
             </div>
         </div>
     );
 }
 
```

## Token Access in React Components

Tokens are no longer directly accessible inside React components. For example, this is no longer valid:

```diff
-const { tokens } = useOidc({ assert: "user logged in" });
-console.log(tokens!.accessToken);
```

There is **no direct replacement** for this, because tokens do not belong in the rendering layer.

What you usually need in the UI is **user information**, which remains available via the decoded ID token:

```tsx
function HeaderButtonLoggedIn() {
    const { decodedIdToken } = useOidc({ assert: "user logged in" });
    return <div>Hello {decodedIdToken.name}</div>;
}
```

The `access_token` should be treated as **opaque**, used only when making authorized API calls.\
Here is an example of correct usage:

{% code title="Custom fetch function that automatically adds the Authorization header" %}
```typescript
export const fetchWithAuth: typeof fetch = async (input, init) => {
    const oidc = await getOidc();

    if (oidc.isUserLoggedIn) {
        const { accessToken } = await oidc.getTokens();

        const headers = new Headers(init?.headers);
        headers.set("Authorization", `Bearer ${accessToken}`);
        (init ??= {}).headers = headers;
    }

    return fetch(input, init);
};
```
{% endcode %}

If you truly need to expose the `access_token` inside a React component, see this custom hook example:\
[Example on GitHub](https://github.com/keycloakify/oidc-spa/blob/d1eede4e0cec6b475a3dbc2b4677b6a02326c05d/examples/tanstack-router-file-based/src/routes/protected.tsx#L120-L174)

***

## Other API Changes

* `__unsafe_ssoSessionIdleSeconds` was renamed to `idleSessionLifetimeInSeconds`.
* `transformUrlBeforeRedirect` now receives a structured object as parameter.

```diff
 createReactOidc({
-    __unsafe_ssoSessionIdleSeconds: 60 * 10,
+    idleSessionLifetimeInSeconds: 60 * 10,

-    transformUrlBeforeRedirect: url => `${url}&ui_locale=fr`,
+    transformUrlBeforeRedirect: ({ authorizationUrl, isSilent }) => {
+        if (isSilent) {
+            return authorizationUrl;
+        }
+        return `${authorizationUrl}&ui_locale=fr`;
+    }
 })
```

***
