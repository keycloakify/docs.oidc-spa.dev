# Migration from keycloak-js

oidc-spa can act as a dropin replacement for keycloak-js.  \
This is usefull if you want to migrate an app currently on keycloak-js without having to go through a painfull migration process.  \
\
Why migrate: &#x20;

-Better security, oidc-spa protect your tokens better againse XSS and suply chain attack.&#x20;

-Better performance, the early init enables for much faster startup time.

-Better UX for your user: Out of the box management of session autologout, avoiding infinite loop when navigating back from login pages. \
\


What you need to port is only the initialization process. &#x20;

keycloak-js support multiple legacy authentication flow that no longer align with current best practice so we're not going to support them in oidc-spa. Namely the implicit and hybrid flow, enableling to not use PKCE. \


```typescript

import Keycloak from "keycloak-js";

const keycloak = 

```
