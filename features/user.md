# The User Object

{% hint style="warning" %}
**Coming in oidc-spa v10.3.** This feature has not been released yet.
{% endhint %}

## Access the User in Your App

Once configured, your custom user model is available wherever you write frontend code.

{% tabs %}
{% tab title="Framework agnostic" %}
{% code title="src/greeting.ts" overflow="wrap" %}
```typescript
import { prOidc } from "./oidc";

export async function getGreeting() {
    const oidc = await prOidc;

    if (!oidc.isUserLoggedIn) {
        return "Hello!";
    }

    const { user } = await oidc.getUser();

    return `Hello ${user.displayName}!`;
}
```
{% endcode %}
{% endtab %}

{% tab title="React" %}
In a component where login is already enforced:

{% code title="src/components/Greeting.tsx" overflow="wrap" %}
```tsx
import { useOidc } from "../oidc";

export function Greeting() {
    const { user } = useOidc({ assert: "user logged in" });

    return <p>Hello {user.displayName}!</p>;
}
```
{% endcode %}

Outside a React component, in browser code:

{% code title="src/greeting.ts" overflow="wrap" %}
```typescript
import { getOidc } from "./oidc";

export async function getGreeting() {
    const oidc = await getOidc({ assert: "user logged in" });
    const { user } = await oidc.getUser();

    return `Hello ${user.displayName}!`;
}
```
{% endcode %}
{% endtab %}

{% tab title="Angular" %}
{% hint style="info" %}
The user abstraction is not available in the Angular adapter yet.
{% endhint %}
{% endtab %}
{% endtabs %}

The `user` object is your application's model of the signed-in person. Shape it around what the **frontend needs to render**.

## Implement `createUser`

To use the user abstaction you need to:

1\) Define User type, you shape it. Define it to contains all the information needed by your UI to render the app.

2\) Implement a function that construct that user, gathering identity information from different possible sources.&#x20;

The user's informations can be retrieved from different sources, here are different focussed examples:&#x20;

{% tabs %}
{% tab title="ID token" %}
The payload of the ID token is the most natural way to get user informations.

{% code title="src/oidc.user.ts" overflow="wrap" %}
```typescript
import type { CreateUser } from "oidc-spa/core";
import { z } from "zod";

export type User = {
    displayName: string;
    email: string | undefined;
    avatarUrl: string | undefined;
};

// This is the expected shape of the decoded ID token.
// This varies from provider to provider (Keycloak, Auth0, EntraID...)
// Replace the schema with the claim shape guaranteed by your provider.
// the amout of information present will depend of the scopes you have required.
const DecodedIdTokenSchema = z.object({
    name: z.string(),
    email: z.string().optional(),
    picture: z.string().optional()
});

export const createUser: CreateUser<User> = ({ 
    decodedIdToken: decodedIdToken_generic 
}) => {
    const decodedIdToken = DecodedIdTokenSchema.parse(decodedIdToken_generic);

    return {
        displayName: decodedIdToken.name,
        email: decodedIdToken.email,
        avatarUrl: decodedIdToken.picture
    };
};
```
{% endcode %}
{% endtab %}

{% tab title="Your API" %}
A dedicated endpoint is often the best option when the model depends on your application's database.

{% code title="src/oidc.user.ts" overflow="wrap" %}
```typescript
import type { CreateUser } from "oidc-spa/core";
import { z } from "zod";

const UserFromApi = z.object({
    id: z.string(),
    displayName: z.string(),
    avatarUrl: z.string().url().optional(),
    canManageBilling: z.boolean()
});

export type User = z.infer<typeof UserFromApi>;

export const createUser: CreateUser<User> = async () => {

    const { fetchWithAuth } = await import("./oidc");
    
    const response = await fetchWithAuth("/api/user");

    if (!response.ok) {
        throw new Error(`GET /api/user failed with ${response.status}`);
    }

    return UserFromApi.parse(await response.json());
};
```
{% endcode %}

The backend can identify the caller from the validated access token, load the corresponding record, and return an application-specific object.
{% endtab %}

{% tab title="JWT access token" %}
Some providers expose roles or groups only in a JWT access token. This Keycloak-shaped example turns those roles into UI-friendly fields.

{% hint style="warning" %}
OAuth clients should normally treat access tokens as opaque. Use this pattern only when your provider documents that the access token is a JWT with a stable claim shape. `decodeJwt()` decodes the payload; it does **not** validate the token. Never use this client-side result to protect backend data.
{% endhint %}

{% code title="src/oidc.user.ts" overflow="wrap" %}
```typescript
import type { CreateUser } from "oidc-spa/core";
import { decodeJwt } from "oidc-spa/decode-jwt";
import { z } from "zod";

export type User = {
    displayName: string;
    roles: string[];
    canSeeAdminNavigation: boolean;
};

const DecodedIdToken = z.object({
    name: z.string()
});

// Replace the schema with the claim shape guaranteed by your provider.
const DecodedAccessToken = z.object({
    realm_access: z
        .object({
            roles: z.array(z.string())
        })
        .optional()
});

export const createUser: CreateUser<User> = ({
    decodedIdToken: decodedIdToken_generic,
    accessToken
}) => {
    const decodedIdToken = DecodedIdToken.parse(decodedIdToken_generic);
    const decodedAccessToken = DecodedAccessToken.parse(decodeJwt(accessToken));
    const roles = decodedAccessToken.realm_access?.roles ?? [];

    return {
        id: decodedIdToken.sub,
        displayName: decodedIdToken.name,
        roles,
        canSeeAdminNavigation: roles.includes("realm-admin")
    };
};
```
{% endcode %}
{% endtab %}

{% tab title="UserInfo" %}
`fetchUserInfo()` calls the standard OIDC UserInfo endpoint discovered from your provider's metadata and attaches the current access token.

{% code title="src/oidc.user.ts" overflow="wrap" %}
```typescript
import type { CreateUser } from "oidc-spa/core";
import { z } from "zod";

export type User = {
    id: string;
    displayName: string;
    email: string | undefined;
    avatarUrl: string | undefined;
};

const UserInfo = z.object({
    sub: z.string(),
    name: z.string(),
    email: z.string().email().optional(),
    picture: z.string().url().optional()
});

export const createUser: CreateUser<User> = async ({ fetchUserInfo }) => {
    const userInfo = UserInfo.parse(await fetchUserInfo());

    return {
        id: userInfo.sub,
        displayName: userInfo.name,
        email: userInfo.email,
        avatarUrl: userInfo.picture
    };
};
```
{% endcode %}

No UserInfo request is made unless you call `fetchUserInfo()`. The available claims still depend on your requested scopes and provider configuration.
{% endtab %}

{% tab title="Keycloak profile" %}
Provider-specific APIs can expose information that is not available through standard OIDC claims. For Keycloak, oidc-spa includes a typed helper for the account profile.

{% code title="src/oidc.user.ts" overflow="wrap" %}
```typescript
import type { CreateUser } from "oidc-spa/core";
import { createKeycloakUtils, isKeycloak } from "oidc-spa/keycloak";

export type User = {
    username: string;
    displayName: string;
    email: string | undefined;
};

export const createUser: CreateUser<User> = async ({ 
    decodedIdToken, 
    accessToken, 
    issuerUri 
}) => {

    const keycloakUtils = createKeycloakUtils({ issuerUri });

    const profile = await keycloakUtils.fetchUserProfile({ accessToken });


    return {
        username: profile.username ?? decodedIdToken.sub,
        displayName: [profile.firstName, profile.lastName].join(" "),
        email: profile.email
    };
};
```
{% endcode %}

The endpoint must be enabled and accessible to your client. Keep this provider-specific code inside `createUser` so the rest of the UI remains provider-agnostic.
{% endtab %}
{% endtabs %}

<details>

<summary>What else is available to <code>createUser</code>?</summary>

* `decodedIdToken` is the raw decoded ID token payload. Validate the claims your UI depends on, even if you also configured `withExpectedDecodedIdTokenShape()`.
* `accessToken` is the current access token. Use it to call a resource server, but do not store it in `User` because tokens rotate.
* `fetchUserInfo` lazily calls the discovered standard UserInfo endpoint.
* `issuerUri` lets you select provider-specific behavior.
* `user_current` is `undefined` on the first build and contains the previous model during subsequent rebuilds.

`createUser` may return a `User` directly or a promise. Do not call `getUser()` directly or indirectly from inside it: `getUser()` is already waiting for `createUser()` to finish.

</details>

## Connect It to Your Adapter

`createUser` is framework-independent. Register it once when configuring oidc-spa.

{% tabs %}
{% tab title="Framework agnostic" %}
<pre class="language-typescript" data-title="src/oidc.ts" data-overflow="wrap"><code class="lang-typescript">import { createOidc } from "oidc-spa/core";
<strong>import { createUser } from "./oidc.user";
</strong>
export const prOidc = createOidc({
    issuerUri: import.meta.env.VITE_OIDC_ISSUER_URI,
    clientId: import.meta.env.VITE_OIDC_CLIENT_ID,
<strong>    createUser
</strong>});
</code></pre>
{% endtab %}

{% tab title="React SPA" %}
<pre class="language-typescript" data-title="src/oidc.ts" data-overflow="wrap"><code class="lang-typescript">import { oidcSpa } from "oidc-spa/react-spa";
<strong>import { createUser, type User } from "./oidc.user";
</strong>
export const { useOidc, getOidc, /*...*/ } = oidcSpa
<strong>    .withUser&#x3C;User>({ createUser })
</strong>    .createUtils();

</code></pre>
{% endtab %}

{% tab title="TanStack Start" %}
<pre class="language-typescript" data-title="src/oidc.ts" data-overflow="wrap"><code class="lang-typescript">import { oidcSpa } from "oidc-spa/react-tanstack-start";
<strong>import { createUser, type User } from "./oidc.user";
</strong>
export const { useOidc, getOidc, /*...*/ } = oidcSpa
<strong>    .withUser&#x3C;User>({ createUser })
</strong>    .createUtils();

</code></pre>

{% hint style="info" %}
In TanStack Start, frontend components and the resource server share a project but not a trust boundary. The app-level `user` is created in the browser for UI concerns. When backend token validation is configured, server functions and API handlers identify and authorize the caller from the validated `oidc.accessTokenClaims` supplied by `oidcFnMiddleware` or `oidcRequestMiddleware`. See [Backend Token Validation](../integration-guides/backend-token-validation/).
{% endhint %}
{% endtab %}

{% tab title="Angular" %}
{% hint style="info" %}
Angular support is not implemented yet, so there is currently no `createUser` registration API.
{% endhint %}
{% endtab %}
{% endtabs %}

## Updating the user

createUser will be called initially by oidc-spa and again every time, upon token rotation, where it detects that the id/access tokens have meaneagfully changed.\
However if you are building the user object from external sources like your API endpoint, if you modify the user, you might need to manually trigger a refresh:  \
\
// TODO: Code block examples.
