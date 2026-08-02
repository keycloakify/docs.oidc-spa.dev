# The User Abstraction

{% hint style="warning" %}
**Coming in oidc-spa v10.3.** This feature has not been released yet.
{% endhint %}

The `user` object is your application's model of the signed-in person. Shape it around what the **frontend needs to render**, rather than mirroring a token or a provider response.

```typescript
export type User = {
    displayName: string;
    avatarUrl: string | undefined;
    canSeeAdminNavigation: boolean;
};
```

Prefer application-level language such as `canSeeAdminNavigation` over provider-specific details such as `realm_access.roles`. This keeps the rest of your UI independent from your identity provider.

{% hint style="warning" %}
The `user` object is for UI decisions, not security. It can determine whether to show an admin link; your resource server must still validate the access token and authorize every request.
{% endhint %}

## Where User Data Comes From

The decoded ID token is the natural starting point: its claims contain identity information the authorization server makes available to your application. In practice, the UI may need information that is stored elsewhere.

| Source                        | Best suited for                                                       | Keep in mind                                            |
| ----------------------------- | --------------------------------------------------------------------- | ------------------------------------------------------- |
| Decoded ID token              | Basic identity claims already returned at sign-in                     | Usually the simplest option                             |
| Your `GET /api/user` endpoint | Application records, preferences, and entitlements from your database | The API must validate the access token                  |
| JWT access token              | Roles or groups that the provider only places in the access token     | Provider-specific and suitable only for UI decisions    |
| OIDC UserInfo endpoint        | Additional standard OIDC claims                                       | Requires a UserInfo endpoint and the appropriate scopes |
| Provider-specific endpoint    | Rich provider data, such as the Keycloak account profile              | Couples the model to that provider                      |

You can use one source or combine several. Fetch only data that the UI actually consumes: extra requests delay construction of the user model.

## Implement `createUser`

Define the model and its constructor in a framework-independent file. Each tab below is a complete, focused alternative; the different `User` shapes are intentional.

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

const IdTokenClaims = z.object({
    sub: z.string(),
    name: z.string(),
    email: z.string().email().optional(),
    picture: z.string().url().optional()
});

export const createUser: CreateUser<User> = ({ decodedIdToken }) => {
    const claims = IdTokenClaims.parse(decodedIdToken);

    return {
        id: claims.sub,
        displayName: claims.name,
        email: claims.email,
        avatarUrl: claims.picture
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

const IdTokenClaims = z.object({
    sub: z.string(),
    name: z.string()
});

const AccessTokenClaims = z.object({
    realm_access: z
        .object({
            roles: z.array(z.string())
        })
        .optional()
});

export const createUser: CreateUser<User> = ({ decodedIdToken, accessToken }) => {
    const identity = IdTokenClaims.parse(decodedIdToken);
    const claims = AccessTokenClaims.parse(decodeJwt(accessToken));
    const roles = claims.realm_access?.roles ?? [];

    return {
        id: identity.sub,
        displayName: identity.name,
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

## Use It With Your Adapter

Connect your `createUser` implementation to the API used by your application, then read the resulting model from frontend code.

{% hint style="info" %}
The user abstraction is currently available with the framework-agnostic API, React SPA adapter, and TanStack Start adapter. Angular support is not implemented yet.
{% endhint %}

{% tabs %}
{% tab title="Framework agnostic" %}

Register the constructor when creating the OIDC client:

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

Then call `getUser()` after checking the authentication state:

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

{% tab title="React SPA" %}

This includes React applications using TanStack Router as a client-side SPA.

Register the constructor before bootstrapping the adapter:

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

In a component where login is already enforced, `user` is available synchronously:

{% code title="src/components/Greeting.tsx" overflow="wrap" %}

```tsx
import { useOidc } from "../oidc";

export function Greeting() {
    const { user } = useOidc({ assert: "user logged in" });

    return <p>Hello {user.displayName}!</p>;
}
```

{% endcode %}

Outside React, in browser code, use `getOidc({ assert: "user logged in" })`, then call `oidc.getUser()`.

{% endtab %}

{% tab title="TanStack Start" %}

Register the constructor before bootstrapping the adapter:

{% code title="src/oidc.ts" overflow="wrap" %}

```typescript
import { oidcSpa } from "oidc-spa/react-tanstack-start";
import { z } from "zod";
import { createUser, type User } from "./oidc.user";

export const { bootstrapOidc, useOidc, getOidc, enforceLogin, oidcFnMiddleware, oidcRequestMiddleware } =
    oidcSpa
        .withUser<User>({ createUser })
        .withAccessTokenValidation({
            type: "RFC 9068: JSON Web Token (JWT) Profile for OAuth 2.0 Access Tokens",
            expectedAudience: ({ process }) => process.env.OIDC_AUDIENCE,
            accessTokenClaimsSchema: z.object({
                sub: z.string(),
                realm_access: z
                    .object({
                        roles: z.array(z.string())
                    })
                    .optional()
            }),
            accessTokenClaims_mock: {
                sub: "mock-user-id",
                realm_access: { roles: ["realm-admin"] }
            }
        })
        .createUtils();

bootstrapOidc(({ process }) => ({
    implementation: "real",
    issuerUri: process.env.OIDC_ISSUER_URI,
    clientId: process.env.OIDC_CLIENT_ID
}));
```

{% endcode %}

Authentication-aware UI is rendered on the client. In a component that may render on the server, first account for the not-ready state.

{% code title="src/components/Greeting.tsx" overflow="wrap" %}

```tsx
import { useOidc } from "../oidc";

export function Greeting() {
    const oidc = useOidc();

    if (!oidc.isOidcReady || !oidc.isUserLoggedIn) {
        return null;
    }

    return <p>Hello {oidc.user.displayName}!</p>;
}
```

{% endcode %}

On a route protected by `beforeLoad: enforceLogin`, you can instead use `useOidc({ assert: "user logged in" })`.

{% endtab %}
{% endtabs %}

{% hint style="info" %}
When the mock implementation signs a user in, provide `user_mock` to `withUser()` or override it in `bootstrapOidc()`. Mock mode returns that object directly and does not call `createUser`. With the framework-agnostic mock API, the equivalent option is named `mockedUser`.
{% endhint %}

## TanStack Start: Frontend User vs. Backend Claims

TanStack Start places frontend components and the resource server in one project, but they remain separate trust boundaries.

| Where the code runs                        | Use                      | Purpose                                                            |
| ------------------------------------------ | ------------------------ | ------------------------------------------------------------------ |
| Browser components and client-only loaders | `user`                   | Render names and avatars, or hide controls the user cannot use     |
| Server functions and API handlers          | `oidc.accessTokenClaims` | Identify the caller and enforce permissions from a validated token |

{% hint style="danger" %}
`user.canSeeAdminNavigation` is a convenience for the frontend. It does not grant access. Enforce the same permission on the server before returning protected data.
{% endhint %}

The TanStack Start adapter runs `createUser()` only in the browser, and `getOidc()` cannot be called on the server. Server functions and API handlers receive a separate OIDC context from middleware.

The TanStack Start setup above validates access tokens and exposes `oidcFnMiddleware` for server functions and `oidcRequestMiddleware` for API handlers:

{% code title="src/routes/admin-data.ts" overflow="wrap" %}

```typescript
import { createServerFn } from "@tanstack/react-start";
import { oidcFnMiddleware } from "../oidc";

export const getAdminData = createServerFn({ method: "GET" })
    .middleware([
        oidcFnMiddleware({
            assert: "user logged in",
            hasRequiredClaims: ({ accessTokenClaims }) =>
                accessTokenClaims.realm_access?.roles.includes("realm-admin")
        })
    ])
    .handler(async ({ context: { oidc } }) => {
        const userId = oidc.accessTokenClaims.sub;

        return { message: `Protected data for ${userId}` };
    });
```

{% endcode %}

Here, the middleware validates the access token and performs the permission check before the handler runs. See [Backend Token Validation](/integration-guides/backend-token-validation) for the complete setup.

## Refresh User Data

oidc-spa caches the constructed model and rebuilds it when its token-claim fingerprint changes. Lifecycle claims such as issued-at and expiry times do not trigger a rebuild. Changes that exist only in your database, UserInfo response, or provider profile cannot be detected automatically.

Call `refreshUser()` after an action that changes remote user data. It renews the tokens, runs `createUser()` again, and returns the new model. For example, with either React adapter:

{% code title="src/useUpdateProfile.ts" overflow="wrap" %}

```typescript
import { useOidc } from "./oidc";
import { updateProfile } from "./api";

export function useUpdateProfile() {
    const { refreshUser } = useOidc({ assert: "user logged in" });

    return async (formData: FormData) => {
        await updateProfile(formData);
        return refreshUser();
    };
}
```

{% endcode %}

With the framework-agnostic API, get the same function from `await oidc.getUser()`.

On rebuilds, `createUser()` receives the previous model as `user_current`. If a later rebuild fails, oidc-spa keeps the previous model. If the first build fails, no user model is produced; React adapters surface this as an initialization error.

For complete projects, see the [TanStack Router SPA example](https://github.com/keycloakify/oidc-spa/tree/main/examples/tanstack-router-file-router) and the [TanStack Start example](https://github.com/keycloakify/oidc-spa/tree/main/examples/tanstack-start).
