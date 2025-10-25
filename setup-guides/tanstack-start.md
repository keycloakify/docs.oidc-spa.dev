---
icon: umbrella-beach
---

# TanStack Start

```bash
npx gitpick keycloakify/oidc-spa/tree/main/examples/tanstack-start start-oidc
cd start-oidc
npm install
npm run dev

# By default, the example runs against Keycloak.
# You can edit the .env file to test other providers.
```

Live example:

{% embed url="https://example-tanstack-start.oidc-spa.dev/" %}

***

## Step-by-step setup

### 1. Install dependencies

{% tabs %}
{% tab title="npm" %}
```bash
npm install oidc-spa zod
```
{% endtab %}

{% tab title="yarn" %}
```bash
yarn add oidc-spa zod
```
{% endtab %}

{% tab title="pnpm" %}
```bash
pnpm add oidc-spa zod
```
{% endtab %}

{% tab title="bun" %}
```bash
bun add oidc-spa zod
```
{% endtab %}
{% endtabs %}

> **Note:**\
> [Zod](https://zod.dev/) is optional but highly recommended.\
> Writing validators manually is error-prone, and skipping validation means losing early guarantees about what your auth server provides.

***

### 2. Configure Vite

Add the plugin to your `vite.config.ts`:

<pre class="language-typescript"><code class="lang-typescript">import { defineConfig } from "vite";
import { tanstackStart } from "@tanstack/react-start/plugin/vite";
import viteReact from "@vitejs/plugin-react";
import viteTsConfigPaths from "vite-tsconfig-paths";
import tailwindcss from "@tailwindcss/vite";
<strong>import { oidcSpa } from "oidc-spa/vite-plugin";
</strong>
export default defineConfig({
  plugins: [
    viteTsConfigPaths({ projects: ["./tsconfig.json"] }),
    tailwindcss(),
    tanstackStart(),
<strong>    oidcSpa(),
</strong>    viteReact(),
  ],
});
</code></pre>

***

### 3. Provide your OIDC environment variables

{% code title=".env" %}
```properties
OIDC_USE_MOCK=false
OIDC_ISSUER_URI=https://cloud-iam.oidc-spa.dev/realms/oidc-spa
OIDC_CLIENT_ID=example-tanstack-start
```
{% endcode %}

You can use our preconfigured Keycloak, Auth0, or Google OAuth test accounts —\
see [this sample file](https://github.com/keycloakify/oidc-spa/blob/main/examples/tanstack-start/.env.sample).

For your own configuration, refer to:

{% content-ref url="../providers-configuration/provider-configuration.md" %}
[provider-configuration.md](../providers-configuration/provider-configuration.md)
{% endcontent-ref %}

***

### 4. Bootstrapping the OIDC API

Create the following file:

{% code title="src/oidc.ts" %}
```typescript
import { oidcSpa } from "oidc-spa/react-tanstack-start";
import { z } from "zod";

export const {
  bootstrapOidc,
  createOidcComponent,
  getOidc,
  enforceLogin,
  oidcFnMiddleware,
  oidcRequestMiddleware,
} = oidcSpa
  .withExpectedDecodedIdTokenShape({
    // Client-side view of the user's identity.
    // You declare what parts of the ID token you plan to use.
    // (You can inspect the console to see the full object.)
    decodedIdTokenSchema: z.object({
      name: z.string(),
      picture: z.string().optional(),
      realm_access: z.object({ roles: z.array(z.string()) }).optional(),
    }),
    decodedIdToken_mock: {
      name: "John Doe",
    },
  })
  .withAccessTokenValidation({
    type: "RFC 9068: JSON Web Token (JWT) Profile for OAuth 2.0 Access Tokens",
    // In oidc-spa, the *frontend* is the OIDC client.
    // The backend (server functions or APIs) plays the role of a
    // resource server — it simply validates access tokens received
    // from the client. It is not part of the authentication flow itself.
    accessTokenClaimsSchema: z.object({
      sub: z.string(), // User ID
      realm_access: z.object({ roles: z.array(z.string()) }).optional(),
    }),
    accessTokenClaims_mock: { sub: "123" },
    expectedAudience: () => "account", // Keycloak default; depends on provider
  })
  .finalize();

// Can be called anywhere (even in React component bodies).
// Only the first call has an effect — subsequent calls are ignored.
// The process object allows accessing environment variables from both
// client and server seamlessly (only the ones you dereference are exposed).
bootstrapOidc(({ process }) =>
  process.env.OIDC_USE_MOCK === "true"
    ? {
        implementation: "mock",
        isUserInitiallyLoggedIn: true,
      }
    : {
        implementation: "real",
        issuerUri: process.env.OIDC_ISSUER_URI,
        clientId: process.env.OIDC_CLIENT_ID,
        debugLogs: true,
      }
);

// A fetch wrapper that automatically adds the Authorization header
// if the user is logged in.
export const fetchWithAuth: typeof fetch = async (input, init) => {
  const oidc = await getOidc();

  if (oidc.isUserLoggedIn) {
    const accessToken = await oidc.getAccessToken();
    const headers = new Headers(init?.headers);
    headers.set("Authorization", `Bearer ${accessToken}`);
    (init ??= {}).headers = headers;
  }

  return fetch(input, init);
};
```
{% endcode %}

***

### 5. Add Header Auth Buttons

<figure><img src="../.gitbook/assets/image (5).png" alt="Auth button example" width="262"><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (4).png" alt="User avatar example" width="196"><figcaption></figcaption></figure>

Code reference:

{% embed url="https://github.com/keycloakify/oidc-spa/blob/e73f63a6cb70753110c0d2caab37b0f8a63926e0/examples/tanstack-start/src/components/Header.tsx#L10-L98" %}

Redirecting users directly to the registration page varies by provider —\
see [this Auth0 example](https://github.com/keycloakify/oidc-spa/blob/e73f63a6cb70753110c0d2caab37b0f8a63926e0/examples/tanstack-router-file-based/src/components/Header.tsx#L98-L110).

***

## Making authenticated requests

Enforcing authentication on routes and server functions:\
([source](https://github.com/keycloakify/oidc-spa/blob/main/examples/tanstack-start/src/routes/demo/start.server-funcs.tsx))

```tsx
import { useState } from "react";
import { createFileRoute, useRouter } from "@tanstack/react-router";
import { createServerFn } from "@tanstack/react-start";
import { enforceLogin, oidcFnMiddleware } from "@/oidc";
import Spinner from "@/components/Spinner";
import { getTodosStore } from "@/data/todos";

const getTodos = createServerFn({ method: "GET" })
  .middleware([oidcFnMiddleware({ assert: "user logged in" })])
  .handler(async ({ context: { oidc } }) => {
    const userId = oidc.accessTokenClaims.sub;
    const todosStore = getTodosStore();
    return await todosStore.readTodos({ userId });
  });

const addTodo = createServerFn({ method: "POST" })
  .inputValidator((d: string) => d)
  .middleware([oidcFnMiddleware({ assert: "user logged in" })])
  .handler(async ({ data, context: { oidc } }) => {
    const userId = oidc.accessTokenClaims.sub;
    const todosStore = getTodosStore();
    const todos = await todosStore.readTodos({ userId });
    todos.push({ id: todos.length + 1, name: data });
    await todosStore.updateTodos({ userId, todos });
  });

export const Route = createFileRoute("/demo/start/server-funcs")({
  beforeLoad: enforceLogin,
  component: Home,
  loader: async () => await getTodos(),
  pendingComponent: () => (
    <div className="flex flex-1 items-center justify-center py-16">
      <Spinner />
    </div>
  ),
});

function Home() {
  const router = useRouter();
  const todos = Route.useLoaderData();
  const [newTodoInputValue, setNewTodoInputValue] = useState("");

  const onAddTodoButtonClick = async () => {
    await addTodo({ data: newTodoInputValue });
    setNewTodoInputValue("");
    router.invalidate();
  };

  return (
    <div>
      <ul>
        {todos.map(todo => (
          <li key={todo.id}>
            <span className="text-lg text-white">{todo.name}</span>
          </li>
        ))}
      </ul>
      <div>
        <input
          type="text"
          value={newTodoInputValue}
          onChange={e => setNewTodoInputValue(e.target.value)}
          onKeyDown={e => e.key === "Enter" && onAddTodoButtonClick()}
          placeholder="Enter a new todo..."
        />
        <button
          disabled={newTodoInputValue.trim().length === 0}
          onClick={onAddTodoButtonClick}
        >
          Add todo
        </button>
      </div>
    </div>
  );
}
```

Authenticate an API:

{% embed url="https://github.com/keycloakify/oidc-spa/blob/main/examples/tanstack-start/src/routes/demo/api.todos.ts" %}

Calling an authenticated API:

{% embed url="https://github.com/keycloakify/oidc-spa/blob/main/examples/tanstack-start/src/routes/demo/start.api-request.tsx" %}

Auto Logout overlay:

{% embed url="https://github.com/keycloakify/oidc-spa/blob/main/examples/tanstack-start/src/components/AutoLogoutWarningOverlay.tsx" %}
