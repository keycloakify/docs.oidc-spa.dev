# 🔌 REST API Client

The primary usecase for a library like oidc-spa is to use a REST API with endpoints that requires authentication. &#x20;

Let's see a very basic example:

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
<strong>import { prOidc } from "oidc";
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
</strong><strong>        throw new Error("We made a logic error: The user should be logged in at this point");
</strong><strong>    }
</strong><strong>    
</strong><strong>    config.headers.Authorization = `Bearer ${oidc.getTokens().accessToken}`;
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
Initialize the React adapter of oidc-spa and expose the prOidc object, a promise of the vanilla OIDC API:

<pre class="language-typescript" data-title="src/oidc.ts"><code class="lang-typescript">import { createReactOidc } from "oidc-spa/react";

export const { 
    OidcProvider, 
    useOidc,
<strong>    prOidc
</strong>} = createReactOidc(/* ... */);
</code></pre>

Create a REST API Client that adds the OIDC Access Token as Autorization header to every HTTP request:

<pre class="language-typescript" data-title="src/api.ts"><code class="lang-typescript">import axios from "axios";
<strong>import { prOidc } from "oidc";
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
</strong><strong>        throw new Error("We made a logic error: The user should be logged in at this point");
</strong><strong>    }
</strong><strong>    
</strong><strong>    config.headers.Authorization = `Bearer ${oidc.getTokens().accessToken}`;
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
This example is purposefully very basic to minimize noise but in your App you should consider using [tRPC](https://trpc.io/) (if you have JS backend) and [TanStack Query](https://tanstack.com/query/v3).
{% endhint %}
{% endtab %}
{% endtabs %}
