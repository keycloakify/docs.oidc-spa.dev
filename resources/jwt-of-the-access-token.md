---
description: And why it's not supposed to be read on the client side.
icon: brackets-curly
---

# JWT Of the Access Token

You might be surprised, or even frustrated, that **oidc-spa** only provides the decoded ID token and not the decoded access token.\
This is intentional: the access token is meant to be **opaque** to the client application. It should be used only as an authentication key (e.g., a Bearer token when calling an API). According to the OIDC specification, the access token is not even required to be a JWT.

The good news is that everything you need is usually found in the ID token. If you notice that certain information appears in the access token but not in the ID token, there are two likely reasons:

1. **Identity server policy** – Your identity provider may have an explicit rule stripping those claims from the ID token.
2. **Schema filtering** – [When using `decodedIdTokenSchema` with Zod](https://github.com/keycloakify/oidc-spa/blob/d1eede4e0cec6b475a3dbc2b4677b6a02326c05d/examples/tanstack-router-file-based/src/oidc.tsx#L39-L53), any claims not declared in your schema will be discarded. This can make it seem like the ID token contains fewer claims than it actually does. To see the complete payload, initialize the adapter with `debugLogs: true` and check your console output.
