---
icon: arrows-to-circle
---

# Talking to different APIs (with different access tokens)

By default with oidc-spa when you create a client, you get issued access token to talk to a specific resource server. \
In our model the resource server in question is usually the backend REST API that you've created specifically for this application, or any code that runs on the server in TanStack Start.  \
\
What is important to understand about the model is that, in oidc-spa, your frontend application IS the client. And your backend is mearly a resource server that you query by attaching an access token as Authorization Bearer.  \
(This is by opposition with model like NextAuth where the constitute the application in the Open ID Connect model. ) \
\
So this is all good ant well as long as your app only need to talk with a single resource server.&#x20;

However in frontend centric app, it's often the case that you would want to talk to different APIs (resource server) like for example:&#x20;

* your own REST Api
* Amazon S3
* Vault
* ...\


Of course you could have your backend have service account to those third party service and use your API as a proxy to query thoses services. &#x20;

This is a perfectly valid approach but in some application architecture, you want to keep your backend as light and stateless as possible and centralize your app logic within the frontend code for minimizing the infra cost and maximizing the reactivity or your app.  \
\
The problem is that you cannot use a single acces token to talk to different API. Well you can't but it's not a great security posture to have.  \
An access token caries claims, those claims represent who the user is, who this token is for and what are the permission of the user in the system, example: &#x20;

```
{
   aud: "vault" // This token is for Vault
   sub: "xxxxxx" // The unique idientifier of the user
   groups: ["staff"] // Define the permission of the user on the system (just an exampl)
}
```

First of, almost OAuth API will require a specific audience to be set even if the aud claim can be an array every api may require claim formatted in slightly different way.  \
\
So the solution to talk to diffetent API is not to just send the same access token to every resource server but to configure your idp so that you can request access tokens for different resource server and craft your different access token precisely as each individual API expect them.  \
\
How do we do that in practice with oidc-spa?  \
\
Well, in an ideal work we would implement RFC 8707 Resource Indicators for OAuth 2.0 and oidc-spa would expose something like getAccessToken({ resource: "https://api1.example.com" }) and you would get a token for this specific API.  \
You would be able to declare different API on your idp and what oidc client can request an access token for said API. It's how it work in Auth0.  \
\
The problem is that it's not how it work in Keycloak, keycloak [does not yet implement RFC 8707](https://github.com/keycloak/keycloak/discussions/35743) and since Keycloak is the defacto standard implementation open id connect server we don't want to support anything that wouldn't be supported by Keycloak.  \
\
In Keycloak's interpretation of the oidc protocol, when you declare an oidc client. You're not actually declaring an oidc client, you're declaring a couple: a client application that can talk to a given resource server.  \
So, if you want to talk with different resource server you need to create different clients (in the same SSO realm) that all point to the same Valid Redirect URI:

* clientId: "myapp", valid redirect uri: "https://myapp.my-company.com/"
* clientId: "myapp-vault": valid redirect uri: "https://myapp.my-company.com/"
* clientId: "myapp-s3": valid redirect uri: "https://myapp.my-company.com/"

And so on. \
For each of those client you can configure protocol mapper to make sure that the access token issuer match the expectation of the resource server.  \
What is regretable is that the limitation of Keycloak means that even if you're not using keycloak and your idp allow you to declare API independently of oidc client, you will still have to to declare things this way so it works with oidc-spa.  \
\
Now that you have created your different clients you can instanciate them simultaneously in your app, oidc-spa has full support for multi client.  \
\
Let's consider the implementation of a "My Secrets" page that would query Hashi corp Vault to access the users secrets:\
We make the example with a react SPA but the important bit are using oidc-spa core so you should be able to infer what it should look in your framwork of choice.

{% code title="src/oidc.ts" %}
```typescript
import { oidcSpa } from "oidc-spa/react-spa";

export const { bootstrapOidc, useOidc, getOidc, enforceLogin } = oidcSpa.finalize();

bootstrapOidc({
    implementation: "real",
    issuerUri: "https://auth.my-company.com/realms/myrealm",
    clientId: "myapp",
    // See note below
    // sessionRestorationMethod: "iframe"
});
```
{% endcode %}

{% code title="app/routes/my-secrets.tsx" %}
```tsx
import { useLoaderData } from "react-router";
import type { Route } from "./+types/protected";

import { getOidc, enforceLogin } from "~/oidc";
// Use the core API directly because this route does not need framework helpers.
import { createOidc } from "oidc-spa/core";

let cache: { oidcAccessToken_vault: string; vaultToken: string } | undefined = undefined;

export async function clientLoader(params: Route.ClientLoaderArgs) {
    // Ensure the user session is already established with the IdP.
    await enforceLogin(params);

    // Initialize the Vault-specific OIDC client.
    // The helper memoizes instances per issuer/client pair, so this is effectively cached.
    const { getTokens: getOidcTokens_vault } = await createOidc({
        // Reuse the same realm as the primary client.
        issuerUri: (await getOidc()).issuerUri,
        // Use the dedicated public client registered for Vault access.
        clientId: "myapp-vault",
        // Silent login works because both clients share the same realm session.
        // The user won't be redirected to the login pages.  
        autoLogin: true,
        // See note below
        // sessionRestorationMethod: "iframe"
    });

    // Retrieve the access token issued for the Vault client.
    // We get a freshly requested one or one that we had in cache,
    // oidc-spa arbitrate.  
    const { accessToken: oidcAccessToken_vault } = await getOidcTokens_vault();

    const vaultBaseUrl = "https://vault.example.com";

    // Exchange the OIDC access token for a Vault token.
    const vaultToken = await (async () => {
        if (cache?.oidcAccessToken_vault === oidcAccessToken_vault) {
            return cache.vaultToken;
        }

        const vaultToken = await fetch(`${vaultBaseUrl}/v1/auth/jwt/login`, {
            body: JSON.stringify({
                role: "web-app",
                jwt: oidcAccessToken_vault
            })
        })
            .then(r => r.json())
            .then(o => o.auth.client_token);

        cache = {
            oidcAccessToken_vault,
            vaultToken
        };

        return vaultToken;
    })();

    // Fetch the caller’s secret values using the Vault token.
    const userSecrets = await fetch(`${vaultBaseUrl}/v1/secret/data/users/me`, {
        headers: {
            "X-Vault-Token": vaultToken
        }
    })
        .then(r => r.json())
        .then(o => o.data.data);

    return { userSecrets };
}

export default function Protected() {
    const { userSecrets } = useLoaderData<typeof clientLoader>();

    return (
        <dl>
            {Object.entries(userSecrets).map(([key, value]) => (
                <div key={key} className="space-y-1">
                    <dt>{key}</dt>
                    <dd>{value}</dd>
                </div>
            ))}
        </dl>
    );
}
```
{% endcode %}

And that's it!  \
\
One important caviat is about developement experience and security.  \
The first time you call createOidc() you'll get a full page redirect if silent session restoration via iframe is not available.  \
This is always the case by default when you use run your app in localhost with the devlopement server.  \
On top of that the moment you have more than one client, if iframe session restoration is not possible, oidc-spa will start persisting the token in session storage to avoid cyclical reaload, which downgrade the security guarenties that oidc-spa offers by default.  \
To remediate that: \
\- make sure that the authorization endpoint of your idp is on the same root domain as your app

* in devloppement, add an exception in your browser to allow it to set third party cookies and set sessionRestorationMethod to "iframe" both in bootstrapOidc and createOidc()\


Detailed instruction here:

{% content-ref url="resources/third-party-cookies-and-session-restoration.md" %}
[third-party-cookies-and-session-restoration.md](resources/third-party-cookies-and-session-restoration.md)
{% endcontent-ref %}

\
