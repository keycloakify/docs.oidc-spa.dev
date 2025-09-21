---
icon: plug
---

# Web API

The primary usecase for a library like oidc-spa is to use it to authenticate against a REST, tRPC, or Websocket API.

## Client Side

Let's see a very basic REST API example:

{% tabs %}
{% tab title="Vanilla API" %}
Initialize oidc-spa and expose the oidc instance as a promise:

{% code title="src/oidc.ts" %}
```typescript
import { createOidc } from "oidc-spa";

export const prOidc = createOidc({/* ... */});
```
{% endcode %}

Create a REST API Client that adds the OIDC Access Token as Autorization header to every HTTP request:

<pre class="language-typescript" data-title="src/api.ts"><code class="lang-typescript">import axios from "axios";
<strong>import { prOidc } from "./oidc";
</strong>
type Api = {
    getTodos: () => Promise&#x3C;{ id: number; title: string; }[]>;
    addTodo: (todo: { title: string; }) => Promise&#x3C;void>;
};

const axiosInstance = axios.create({ baseURL: import.meta.env.API_URL });

axiosInstance.interceptors.request.use(async config => {

<strong>    const oidc= await prOidc;
</strong><strong>
</strong><strong>    if( !oidc.isUserLoggedIn ){
</strong><strong>        throw new Error("We made a logic error: If the user isn't logged in we shouldn't be making request to an API endpoint that requires authentication");
</strong><strong>    }
</strong><strong>    
</strong><strong>    const { accessToken } = await oidc.getTokens();
</strong><strong>    
</strong><strong>    config.headers.Authorization = `Bearer ${accessToken}`;
</strong>
    return config;

});

export const api: Api = {
    getTodo: ()=> axiosInstance.get("/todo").then(response => response.data),
    addTodo: todo => axiosInstance.post("/todo", todo).then(response => response.data)
};
</code></pre>
{% endtab %}

{% tab title="React API" %}
<pre class="language-typescript" data-title="src/oidc.ts"><code class="lang-typescript">import { createReactOidc } from "oidc-spa/react";

export const { 
    OidcProvider, 
    useOidc,
<strong>    getOidc
</strong>} = createReactOidc(/* ... */);
</code></pre>

Create a REST API Client that adds the OIDC Access Token as Autorization header to every HTTP request:

<pre class="language-typescript" data-title="src/api.ts"><code class="lang-typescript">import axios from "axios";
<strong>import { getOidc } from "./oidc";
</strong>
type Api = {
    getTodos: () => Promise&#x3C;{ id: number; title: string; }[]>;
    addTodo: (todo: { title: string; }) => Promise&#x3C;void>;
};

const axiosInstance = axios.create({ baseURL: import.meta.env.API_URL });

axiosInstance.interceptors.request.use(async config => {

<strong>    const oidc= await getOidc();
</strong><strong>
</strong><strong>    if( !oidc.isUserLoggedIn ){
</strong><strong>        throw new Error("We made a logic error: The user should be logged in at this point");
</strong><strong>    }
</strong><strong>    
</strong><strong>    const { accessToken } = await oidc.getTokens();
</strong><strong>    
</strong><strong>    config.headers.Authorization = `Bearer ${accessToken}`;
</strong>
    return config;

});

export const api: Api = {
    getTodo: ()=> axiosInstance.get("/todo").then(response => response.data),
    addTodo: todo => axiosInstance.post("/todo", todo).then(response => response.data)
};
</code></pre>

Using your REST API client in your REACT components: &#x20;

<pre class="language-tsx"><code class="lang-tsx"><strong>import { api } from "../api";
</strong>
type Todo= {
    id: number; 
    title: string;
};

function UserTodos(){

    const [todos, setTodos] = useState&#x3C;Todo[] | undefined>(undefined);

    useEffect(
        ()=> {
<strong>            api.getTodos().then(todos => setTodos(todos));
</strong>        },
        []
    );

    if(todos === undefined){
        return &#x3C;>Loading your todos items ⌛️&#x3C;/>
    }

    return (
        &#x3C;ul>
            {todos.map(todo => (
                &#x3C;li key={todo.id}>{todo.title}&#x3C;/li>
            ))}
        &#x3C;/ul>
    );

}
</code></pre>

{% hint style="info" %}
This example is purposefully very basic to minimize noise but in your App you might want to consider using solutions like [tRPC](https://trpc.io/) (if you have JS backend) and [TanStack Query](https://tanstack.com/query/v3).
{% endhint %}
{% endtab %}
{% endtabs %}

## Server Side

If you're implementing a JavaScript Backend (Node/Deno/webworker) `oidc-spa` also exposes an utility to help you validate and decode the access token that your client sends in the authorization header.  \
Granted, this is fully optional feel free to use anything else.  \
\
Let's assume we have a Node.js REST API build with Express or Hono.  \
You can create an oidc file as such:

{% code title="src/oidc.ts" %}
```typescript
import { createOidcBackend } from "oidc-spa/backend";
import { z } from "zod";
import { HTTPException } from "hono/http-exception";

const zDecodedAccessToken = z.object({
    sub: z.string(),
    aud: z.union([z.string(), z.array(z.string())]),
    realm_access: z.object({
        roles: z.array(z.string())
    })
    // Some other info you might want to read from the accessToken, example:
    // preferred_username: z.string()
});

export type DecodedAccessToken = z.infer<typeof zDecodedAccessToken>;

export async function createDecodeAccessToken(params: { 
    issuerUri: string;
    audience: string 
}) {
    const { issuerUri, audience } = params;

    const { verifyAndDecodeAccessToken } = await createOidcBackend({
        issuerUri,
        decodedAccessTokenSchema: zDecodedAccessToken
    });

    function decodeAccessToken(params: {
        authorizationHeaderValue: string | undefined;
        requiredRole?: string;
    }): DecodedAccessToken {
        const { authorizationHeaderValue, requiredRole } = params;

        if (authorizationHeaderValue === undefined) {
            throw new HTTPException(401);
        }

        const result = verifyAndDecodeAccessToken({
            accessToken: authorizationHeaderValue.replace(/^Bearer /, "")
        });

        if (!result.isValid) {
            switch (result.errorCase) {
                case "does not respect schema":
                    throw new Error(`The access token does not respect the schema ${result.errorMessage}`);
                case "invalid signature":
                case "expired":
                    throw new HTTPException(401);
            }
        }

        const { decodedAccessToken } = result;

        if (requiredRole !== undefined && !decodedAccessToken.realm_access.roles.includes(requiredRole)) {
            throw new HTTPException(401);
        }

        {
            const { aud } = decodedAccessToken;

            const aud_array = typeof aud === "string" ? [aud] : aud;

            if (!aud_array.includes(audience)) {
                throw new HTTPException(401);
            }
        }

        return decodedAccessToken;
    }

    return { decodeAccessToken };
}
```
{% endcode %}

Then you can enforce that some endpoints of your API requires the user to be authenticated, in this example we use Hono:&#x20;

<pre class="language-typescript"><code class="lang-typescript">import { z, createRoute, OpenAPIHono } from "@hono/zod-openapi";
import { serve } from "@hono/node-server"
import { HTTPException } from "hono/http-exception";
import { getUserTodoStore } from "./todo";
<strong>import { createDecodeAccessToken } from "./oidc";
</strong>
(async function main() {

<strong>    const { decodeAccessToken } = await createDecodeAccessToken({
</strong><strong>        // Here example with Keycloak but it work the same with 
</strong><strong>        // any provider as long as the access token is a JWT.
</strong><strong>        issuerUri: "https://auth.my-company.com/realms/myrealm",
</strong><strong>        audience: "account" // default audience in Keycloak
</strong><strong>    });
</strong>
    const app = new OpenAPIHono();

    {

        const route = createRoute({
            method: 'get',
            path: '/todos',
            responses: {/* ... */}
        });

        app.openapi(route, async c => {

<strong>            const decodedAccessToken = decodeAccessToken({
</strong><strong>                authorizationHeaderValue: c.req.header("Authorization")
</strong><strong>            });
</strong>
            const todos = await getUserTodoStore(decodedAccessToken.sub).getAll();

            return c.json(todos);

        });

    }

    const port = parseInt(process.env.PORT);

    serve({
        fetch: app.fetch,
        port
    })

    console.log(`\nServer running. OpenAPI documentation available at http://localhost:${port}/doc`)

})();
</code></pre>

## Testable example

{% content-ref url="setup-guides/tanstack-router-+-node-rest-api.md" %}
[tanstack-router-+-node-rest-api.md](setup-guides/tanstack-router-+-node-rest-api.md)
{% endcontent-ref %}
