# The User Abstraction

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

The React API is shared by the React SPA and TanStack Start adapters.

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

The `user` object is your application's model of the signed-in person. Shape it around what the **frontend needs to render**, rather than mirroring a token or provider response. Prefer application-level fields such as `canSeeAdminNavigation` over provider-specific details such as `realm_access.roles`.

{% hint style="warning" %}
The `user` object is for UI decisions, not security. It can determine whether to show an admin link; your resource server must still validate the access token and authorize every request.
{% endhint %}

## Implement `createUser`

`createUser` translates identity data into the app-specific `User` consumed by your UI. The decoded ID token is the natural starting point, but you can use another source when it does not contain everything the UI needs.

Each tab below is a complete, focused alternative. Combine sources only when your model requires it; the different `User` shapes are intentional.

{% tabs %}
{% tab title="ID token" %}

Use the ID token when it already contains everything your UI needs.

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

const DecodedIdToken = z.object({
    sub: z.string(),
    name: z.string(),
    email: z.string().email().optional(),
    picture: z.string().url().optional()
});

export const createUser: CreateUser<User> = ({ decodedIdToken: decodedIdToken_generic }) => {
    const decodedIdToken = DecodedIdToken.parse(decodedIdToken_generic);

    return {
        id: decodedIdToken.sub,
        displayName: decodedIdToken.name,
        email: decodedIdToken.email,
        avatarUrl: decodedIdToken.picture
    };
};
```

{% endcode %}

The schema is the contract between your UI and your provider. Request the required scopes and configure the provider to include these claims; for example, email commonly requires the `email` scope.

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

export const createUser: CreateUser<User> = async ({ accessToken }) => {
    const response = await fetch("/api/user", {
        headers: {
            Authorization: `Bearer ${accessToken}`
        }
    });

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
    id: string;
    displayName: string;
    roles: string[];
    canSeeAdminNavigation: boolean;
};

const DecodedIdToken = z.object({
    sub: z.string(),
    name: z.string()
});

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

Replace the schema with the claim shape guaranteed by your provider.

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
    id: string;
    username: string;
    displayName: string;
    email: string | undefined;
};

export const createUser: CreateUser<User> = async ({ decodedIdToken, accessToken, issuerUri }) => {
    if (!isKeycloak({ issuerUri })) {
        throw new Error("This user model requires Keycloak");
    }

    const profile = await createKeycloakUtils({ issuerUri }).fetchUserProfile({ accessToken });
    const fullName = [profile.firstName, profile.lastName].filter(Boolean).join(" ");

    return {
        id: profile.id,
        username: profile.username ?? decodedIdToken.sub,
        displayName: fullName || profile.username || decodedIdToken.sub,
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

-   `decodedIdToken` is the raw decoded ID token payload. Validate the claims your UI depends on, even if you also configured `withExpectedDecodedIdTokenShape()`.
-   `accessToken` is the current access token. Use it to call a resource server, but do not store it in `User` because tokens rotate.
-   `fetchUserInfo` lazily calls the discovered standard UserInfo endpoint.
-   `issuerUri` lets you select provider-specific behavior.
-   `user_current` is `undefined` on the first build and contains the previous model during subsequent rebuilds.

`createUser` may return a `User` directly or a promise. Do not call `getUser()` directly or indirectly from inside it: `getUser()` is already waiting for `createUser()` to finish.

</details>

## Connect It to Your Adapter

`createUser` is framework-independent. Register it once when configuring oidc-spa.

{% tabs %}
{% tab title="Framework agnostic" %}

{% code title="src/oidc.ts" overflow="wrap" %}

```typescript
import { createOidc } from "oidc-spa/core";
import { createUser } from "./oidc.user";

export const prOidc = createOidc({
    issuerUri: import.meta.env.VITE_OIDC_ISSUER_URI,
    clientId: import.meta.env.VITE_OIDC_CLIENT_ID,
    createUser
});
```

{% endcode %}

{% endtab %}

{% tab title="React SPA" %}

{% code title="src/oidc.ts" overflow="wrap" %}

```typescript
import { oidcSpa } from "oidc-spa/react-spa";
import { createUser, type User } from "./oidc.user";

export const { bootstrapOidc, useOidc, getOidc, enforceLogin, OidcInitializationGate } = oidcSpa
    .withUser<User>({ createUser })
    .createUtils();

bootstrapOidc({
    implementation: "real",
    issuerUri: import.meta.env.VITE_OIDC_ISSUER_URI,
    clientId: import.meta.env.VITE_OIDC_CLIENT_ID
});
```

{% endcode %}

{% endtab %}

{% tab title="TanStack Start" %}

{% code title="src/oidc.ts" overflow="wrap" %}

```typescript
import { oidcSpa } from "oidc-spa/react-tanstack-start";
import { createUser, type User } from "./oidc.user";

export const { bootstrapOidc, useOidc, getOidc, enforceLogin } = oidcSpa
    .withUser<User>({ createUser })
    .createUtils();

bootstrapOidc(({ process }) => ({
    implementation: "real",
    issuerUri: process.env.OIDC_ISSUER_URI,
    clientId: process.env.OIDC_CLIENT_ID
}));
```

{% endcode %}

{% hint style="info" %}
In TanStack Start, frontend components and the resource server share a project but not a trust boundary. The app-level `user` is created in the browser for UI concerns. When backend token validation is configured, server functions and API handlers identify and authorize the caller from the validated `oidc.accessTokenClaims` supplied by `oidcFnMiddleware` or `oidcRequestMiddleware`. See [Backend Token Validation](/integration-guides/backend-token-validation).
{% endhint %}

{% endtab %}

{% tab title="Angular" %}

{% hint style="info" %}
Angular support is not implemented yet, so there is currently no `createUser` registration API.
{% endhint %}

{% endtab %}
{% endtabs %}

{% hint style="info" %}
oidc-spa calls `createUser()` again when relevant claims change in the decoded ID token or a decodable JWT access token. If data changes without a token-claim change—for example in `/api/user`, UserInfo, or a provider profile—call `refreshUser()` to force a rebuild. It is available from `useOidc()` in React and from `oidc.getUser()` in the framework-agnostic API.
{% endhint %}

For complete projects, see the [TanStack Router SPA example](https://github.com/keycloakify/oidc-spa/tree/main/examples/tanstack-router-file-router) and the [TanStack Start example](https://github.com/keycloakify/oidc-spa/tree/main/examples/tanstack-start).
