---
icon: umbrella-beach
---

# TanStack Start

```bash
npx gitpick keycloakify/oidc-spa/tree/main/examples/tanstack-start start-oidc
cd start-oidc
npm install
npm dev

# By default the example runs agains Keycloak, you can edit the .env file
# to test with other providers.
```

The example is live here:&#x20;

{% embed url="https://example-tanstack-start.oidc-spa.dev/" %}

## Step by Step setup

### Install the lib

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

NOTE: [Zod](https://zod.dev/) is optional but recommended as it's cumbersome and error prone to have to write validators manually and not using validator at all makes you loose the ability to check early that the server is providing the information you expect about the user early. &#x20;

### Setup the Vite Plugin

<pre class="language-typescript" data-title="vite.config.ts"><code class="lang-typescript">import { defineConfig } from "vite";
import { tanstackStart } from "@tanstack/react-start/plugin/vite";
import viteReact from "@vitejs/plugin-react";
import viteTsConfigPaths from "vite-tsconfig-paths";
import tailwindcss from "@tailwindcss/vite";
<strong>import { oidcSpa } from "oidc-spa/vite-plugin";
</strong>
const config = defineConfig({
    plugins: [
        viteTsConfigPaths({ projects: ["./tsconfig.json"] }),
        tailwindcss(),
        tanstackStart(),
<strong>        oidcSpa(),
</strong>        viteReact()
    ]
});

export default config;
</code></pre>

### Provide the env required to connect to your auth server

{% code title=".env" %}
```properties
OIDC_USE_MOCK=false
OIDC_ISSUER_URI=https://cloud-iam.oidc-spa.dev/realms/oidc-spa
OIDC_CLIENT_ID=example-tanstack-start
```
{% endcode %}

Feel free to use our Keycloak, Auth0, Google OAuth account to test, you have [some sample here](https://github.com/keycloakify/oidc-spa/blob/main/examples/tanstack-start/.env.sample).

And then refer to our guide to get your own configs:\


{% content-ref url="../providers-configuration/provider-configuration.md" %}
[provider-configuration.md](../providers-configuration/provider-configuration.md)
{% endcontent-ref %}

### Bootstraping the API

Create the folloing file.

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
    oidcRequestMiddleware
} = oidcSpa
    .withExpectedDecodedIdTokenShape({
        // This is the information that you have of the user on the client.
        // NOTE: This is purely declarative you declare what you'll use among
        // the information that the auth server provide about the user.
        // If you're not sure what's available you can open the console
        // and you'll see the full object.
        decodedIdTokenSchema: z.object({
            name: z.string(),
            picture: z.string().optional(),
            realm_access: z.object({ roles: z.array(z.string()) }).optional()
        }),
        decodedIdToken_mock: {
            name: "John Doe"
        }
    })
    .withAccessTokenValidation({
        type: "RFC 9068: JSON Web Token (JWT) Profile for OAuth 2.0 Access Tokens",
        // In oidc-spa, the fronted code is the oidc client. 
        // The backend (server function, API) is not involved in the auth
        // process at all. 
        // It just receive an access_token from the client making API request
        // or calling server function. 
        // The Backend is treated as a resource server in the OIDC model.
        // The access_token, once decoded usually contain the same information
        // than the decoded id_token, but you'll use different claims.
        // For example the name and profile pic of the user is not really usefull
        // on the backend. But we 100% need the user id.
        accessTokenClaimsSchema: z.object({
            sub: z.string(), // This is the user id
            realm_access: z.object({ roles: z.array(z.string()) }).optional()
        }),
        accessTokenClaims_mock: {
            sub: "123"
        },
        // NOTE: By default the audience claim of the access token issued by
        // keycloak is "account" but it depend of the adapter.
        expectedAudience: (/*{ paramsOfBootstrap, process }*/) => "account",
    })
    .finalize();

// Can be call anywhere, even in the body of a React component.
// All subsequent calls will be safely ignored.
// The process object is passed as argument so you can retreive the
// env variable of the server on the client transparently.
// Don't worry only the vars that you dereference will be pulled.
bootstrapOidc(({ process }) =>
    process.env.OIDC_USE_MOCK === "true"
        ? {
              implementation: "mock",
              isUserInitiallyLoggedIn: true
          }
        : {
              implementation: "real",
              issuerUri: process.env.OIDC_ISSUER_URI,
              clientId: process.env.OIDC_CLIENT_ID,
              debugLogs: true
          }
);

// This is the fetch API that automatically attach the access token
// as authorization header if the user is logged in.
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

### Create your Header Auth Buttons

<figure><img src="../.gitbook/assets/image (5).png" alt="" width="262"><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (4).png" alt="" width="196"><figcaption></figcaption></figure>

You can see the code here:

{% embed url="https://github.com/keycloakify/oidc-spa/blob/e73f63a6cb70753110c0d2caab37b0f8a63926e0/examples/tanstack-start/src/components/Header.tsx#L10-L98" %}

How to redirect directly to the register pages varis from provider to provider, [example with Auth0](https://github.com/keycloakify/oidc-spa/blob/e73f63a6cb70753110c0d2caab37b0f8a63926e0/examples/tanstack-router-file-based/src/components/Header.tsx#L98-L110).

## Making authenticated request

Enforcing Auth on a route and using protected server functions ([source here](https://github.com/keycloakify/oidc-spa/blob/main/examples/tanstack-start/src/routes/demo/start.server-funcs.tsx)).

<pre class="language-tsx" data-title="examples/tanstack-start/src/routes/demo/start.server-funcs.tsx"><code class="lang-tsx">import { useState } from "react";
import { createFileRoute, useRouter } from "@tanstack/react-router";
import { createServerFn } from "@tanstack/react-start";
<strong>import { enforceLogin, oidcFnMiddleware } from "@/oidc";
</strong>import Spinner from "@/components/Spinner";

import { getTodosStore } from "@/data/todos";

const getTodos = createServerFn({ method: "GET" })
<strong>    .middleware([oidcFnMiddleware({ assert: "user logged in" })])
</strong>    .handler(async ({ context: { oidc } }) => {
        const userId = oidc.accessTokenClaims.sub;

        const todosStore = getTodosStore();

        return await todosStore.readTodos({ userId });
    });

const addTodo = createServerFn({ method: "POST" })
    .inputValidator((d: string) => d)
<strong>    .middleware([oidcFnMiddleware({ assert: "user logged in" })])
</strong>    .handler(async ({ data, context: { oidc } }) => {
        const userId = oidc.accessTokenClaims.sub;

        const todosStore = getTodosStore();

        const todos = await todosStore.readTodos({ userId });
        todos.push({ id: todos.length + 1, name: data });

        await todosStore.updateTodos({ userId, todos });
    });

export const Route = createFileRoute("/demo/start/server-funcs")({
<strong>    beforeLoad: enforceLogin,
</strong>    component: Home,
    loader: async () => await getTodos(),
    pendingComponent: () => (
        &#x3C;div className="flex flex-1 items-center justify-center py-16">
            &#x3C;Spinner />
        &#x3C;/div>
    )
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
        &#x3C;div>
            &#x3C;ul>
                {todos.map(todo => (
                    &#x3C;li key={todo.id}>
                        &#x3C;span className="text-lg text-white">{todo.name}&#x3C;/span>
                    &#x3C;/li>
                ))}
            &#x3C;/ul>
            &#x3C;div>
                &#x3C;input
                    type="text"
                    value={newTodoInputValue}
                    onChange={e => setNewTodoInputValue(e.target.value)}
                    onKeyDown={e => {
                        if (e.key === "Enter") {
                            onAddTodoButtonClick();
                        }
                    }}
                    placeholder="Enter a new todo..."
                />
                &#x3C;button disabled={newTodoInputValue.trim().length === 0} onClick={onAddTodoButtonClick}>
                    Add todo
                &#x3C;/button>
            &#x3C;/div>
        &#x3C;/div>
    );
}
</code></pre>

Authenticate an API.

{% embed url="https://github.com/keycloakify/oidc-spa/blob/main/examples/tanstack-start/src/routes/demo/api.todos.ts" %}

Calling an authenticated API:

{% embed url="https://github.com/keycloakify/oidc-spa/blob/main/examples/tanstack-start/src/routes/demo/start.api-request.tsx" %}

