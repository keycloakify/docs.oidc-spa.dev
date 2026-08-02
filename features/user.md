# The User Object

{% hint style="warning" %}
**Coming in oidc-spa v10.3.** This feature has not been released yet.
{% endhint %}

## Access the User in Your App

Once configured, your custom user model is available throughout your frontend code.

{% tabs %}
{% tab title="Framework Agnostic" %}
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

In browser code outside a React component:

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

Keep `src/oidc.user.ts` focused on translating identity data into the model consumed by your UI:

1. Define a `User` type containing exactly the information your UI needs.
2. Implement `createUser` to gather that information and return a `User`.
3. If you use mock mode, define a complete `user_mock` with the same shape.

Mock mode uses `user_mock` directly; it does not call `createUser`. The following focused examples are self-contained—choose the source that best fits your application, or combine several of them.

{% tabs %}
{% tab title="ID token" %}
The decoded payload of the ID token is the simplest source when it already contains everything your UI needs.

{% code title="src/oidc.user.ts" overflow="wrap" %}

```typescript
import type { CreateUser } from "oidc-spa/core";
import { z } from "zod";

export type User = {
    displayName: string;
    email: string | undefined;
    avatarUrl: string | undefined;
};

// Match this schema to the claims guaranteed by your provider.
// Available claims depend on its configuration and the requested scopes.
const DecodedIdToken = z.object({
    name: z.string(),
    email: z.string().optional(),
    picture: z.string().optional()
});

export const createUser: CreateUser<User> = ({ decodedIdToken: decodedIdToken_generic }) => {
    const decodedIdToken = DecodedIdToken.parse(decodedIdToken_generic);

    return {
        displayName: decodedIdToken.name,
        email: decodedIdToken.email,
        avatarUrl: decodedIdToken.picture
    };
};

export const user_mock: User = {
    displayName: "John Doe",
    email: "john.doe@example.com",
    avatarUrl: undefined
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

export const user_mock: User = {
    id: "user-123",
    displayName: "John Doe",
    avatarUrl: undefined,
    canManageBilling: true
};
```

{% endcode %}

The backend validates the access token, identifies the caller, loads the corresponding record, and returns an application-specific object.
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
        displayName: decodedIdToken.name,
        roles,
        canSeeAdminNavigation: roles.includes("realm-admin")
    };
};

export const user_mock: User = {
    displayName: "John Doe",
    roles: ["realm-admin"],
    canSeeAdminNavigation: true
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

export const user_mock: User = {
    id: "user-123",
    displayName: "John Doe",
    email: "john.doe@example.com",
    avatarUrl: undefined
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
import { createKeycloakUtils } from "oidc-spa/keycloak";

export type User = {
    username: string;
    displayName: string;
    email: string | undefined;
};

export const createUser: CreateUser<User> = async ({ decodedIdToken, accessToken, issuerUri }) => {
    const keycloakUtils = createKeycloakUtils({ issuerUri });
    const profile = await keycloakUtils.fetchUserProfile({ accessToken });
    const fullName = [profile.firstName, profile.lastName].filter(Boolean).join(" ");

    return {
        username: profile.username ?? decodedIdToken.sub,
        displayName: fullName || profile.username || decodedIdToken.sub,
        email: profile.email
    };
};

export const user_mock: User = {
    username: "john.doe",
    displayName: "John Doe",
    email: "john.doe@example.com"
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

`createUser` is framework-independent. Register it when configuring oidc-spa. If your project supports mock mode, register `user_mock` alongside it.

{% tabs %}
{% tab title="Framework Agnostic" %}
{% code title="src/oidc.ts" overflow="wrap" %}

```typescript
import { createOidc } from "oidc-spa/core";
import { createMockOidc } from "oidc-spa/core-mock";
import { createUser, user_mock } from "./oidc.user";

export const prOidc =
    import.meta.env.VITE_OIDC_USE_MOCK === "true"
        ? createMockOidc({
              isUserInitiallyLoggedIn: true,
              mockedUser: user_mock
          })
        : createOidc({
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
import { createUser, user_mock, type User } from "./oidc.user";

export const { bootstrapOidc, useOidc, getOidc } = oidcSpa
    .withUser<User>({ createUser, user_mock })
    .createUtils();
```

{% endcode %}

When `bootstrapOidc()` receives `{ implementation: "mock", ... }`, it uses this `user_mock` by default. You can override it by passing a different `user_mock` to that bootstrap call.
{% endtab %}

{% tab title="TanStack Start" %}
{% code title="src/oidc.ts" overflow="wrap" %}

```typescript
import { oidcSpa } from "oidc-spa/react-tanstack-start";
import { createUser, user_mock, type User } from "./oidc.user";

export const { bootstrapOidc, useOidc, getOidc } = oidcSpa
    .withUser<User>({ createUser, user_mock })
    .createUtils();
```

{% endcode %}

When `bootstrapOidc()` receives `{ implementation: "mock", ... }`, it uses this `user_mock` by default. You can override it by passing a different `user_mock` to that bootstrap call.

{% hint style="info" %}
In TanStack Start, frontend components and the resource server share a project, but they do not share a trust boundary.

The app-level `user` is created in the browser for UI concerns. On the backend, server functions and API handlers identify and authorize the caller from the validated `oidc.accessTokenClaims` supplied by `oidcFnMiddleware` or `oidcRequestMiddleware`. See [Backend Token Validation](../integration-guides/backend-token-validation/).
{% endhint %}
{% endtab %}

{% tab title="Angular" %}
{% hint style="info" %}
Angular support is not implemented yet, so there is currently no `createUser` registration API.
{% endhint %}
{% endtab %}
{% endtabs %}

## Keep the User Up to Date

`oidc-spa` runs `createUser` when it first builds the signed-in user's model. It runs `createUser` again when it detects meaningful changes in the decoded ID token or the decoded payload of a JWT access token. Routine rotation-only changes, such as new `iat` or `exp` values, do not rebuild the user.

When `createUser` reads from an external source—such as your API, UserInfo, or a provider-specific profile endpoint—`oidc-spa` cannot detect changes to that source. After an update succeeds, call `refreshUser()` to rebuild the user.

With the real implementation, `refreshUser()` renews the tokens, forces `createUser` to run again, notifies user subscribers, and resolves to the refreshed `User`. In mock mode, it simply resolves to the configured `user_mock`.

{% tabs %}
{% tab title="Framework Agnostic" %}
{% code title="src/profile.ts" overflow="wrap" %}

```typescript
import { prOidc } from "./oidc";

export async function refreshCurrentUser() {
    const oidc = await prOidc;

    if (!oidc.isUserLoggedIn) {
        throw new Error("Cannot refresh the user: no user is logged in");
    }

    const { refreshUser } = await oidc.getUser();

    return refreshUser();
}
```

{% endcode %}

Call `refreshCurrentUser()` after the external update has completed.
{% endtab %}

{% tab title="React SPA" %}
{% code title="src/components/SaveProfileButton.tsx" overflow="wrap" %}

```tsx
import { useOidc } from "../oidc";

type Props = {
    saveProfile: () => Promise<void>;
};

export function SaveProfileButton({ saveProfile }: Props) {
    const { refreshUser } = useOidc({ assert: "user logged in" });

    async function onClick() {
        await saveProfile();
        await refreshUser();
    }

    return (
        <button type="button" onClick={onClick}>
            Save profile
        </button>
    );
}
```

{% endcode %}

Components that read `user` through `useOidc()` re-render with the refreshed value.
{% endtab %}

{% tab title="TanStack Start" %}
{% code title="src/components/SaveProfileButton.tsx" overflow="wrap" %}

```tsx
import { useOidc } from "../oidc";

type Props = {
    saveProfile: () => Promise<void>;
};

export function SaveProfileButton({ saveProfile }: Props) {
    const { refreshUser } = useOidc({ assert: "user logged in" });

    async function onClick() {
        await saveProfile();
        await refreshUser();
    }

    return (
        <button type="button" onClick={onClick}>
            Save profile
        </button>
    );
}
```

{% endcode %}

Here, `saveProfile` can call a server function. `refreshUser()` still belongs to the browser-side user abstraction, so call it from frontend code after the server function resolves.
{% endtab %}

{% tab title="Angular" %}
{% hint style="info" %}
The user abstraction is not available in the Angular adapter yet.
{% endhint %}
{% endtab %}
{% endtabs %}
