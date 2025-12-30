---
description: Securing a WebSocket connection
icon: right-left-large
---

# WebSocket

Here’s how to secure a WebSocket connection.\
We’ll review a minimal real-time chat example.\
The server simply echoes back what you send.

This is what we’re building:

{% embed url="https://youtu.be/tEdYRUcAxFA" %}

You can test it live here:

{% embed url="https://vite-insee-starter.demo-domain.ovh/chat" %}

This example uses Node.js + Hono.\
We don’t provide a framework-by-framework (or runtime-by-runtime) guide yet.\
But you should be able to adapt the same approach to your environment.

{% hint style="info" %}
Key takeaways:

* Authentication happens when handling the HTTP upgrade request.
* Browsers don’t let you attach custom headers to a WebSocket upgrade request. Use the `protocols` parameter to carry the access token, then read it server-side from `Sec-WebSocket-Protocol`.
* WebSocket upgrades are **out of scope for DPoP**. There’s no RFC-defined way to send and validate a DPoP proof on the upgrade request. In practice, you must skip DPoP proof validation for the upgrade (`rejectIfAccessTokenDPoPBound: false`). If you need DPoP-grade guarantees on the socket, add an application-level handshake (off-channel).
{% endhint %}

### Server-side code

[Source code](https://github.com/InseeFrLab/todo-rest-api/blob/e00a8a6ed95514c6be4b210506a22b0f0acf24a0/src/main.ts#L36-L53)

<pre class="language-typescript" data-title="src/main.ts"><code class="lang-typescript">import { Hono } from "hono";
<strong>import { createNodeWebSocket } from "@hono/node-ws";
</strong>import { serve } from "@hono/node-server";
import { bootstrapAuth, getUser, getUser_ws } from "./auth"; // See below

function startHonoServer() {

    bootstrapAuth({
        implementation: "real", // or "mock", see: https://docs.oidc-spa.dev/v/v8/integration-guides/backend-token-validation/mock-modes
        issuerUri: process.env.OIDC_ISSUER_URI!,
        expectedAudience: process.env.OIDC_AUDIENCE
    });

    const app = new Hono();

    app.get("/api/todos", async c => { /* ... */ });
    
<strong>    const { injectWebSocket, upgradeWebSocket } = createNodeWebSocket({ app });
</strong>
<strong>    app.get(
</strong><strong>        "/ws",
</strong><strong>        upgradeWebSocket(async c => {
</strong>
<strong>            const user = await getUser_ws({ req: c.req });
</strong>
<strong>            return {
</strong><strong>                onOpen: (_event, ws) => {
</strong><strong>                    ws.send(`Hello ${user.name}`);
</strong><strong>                },
</strong><strong>                onMessage(event, ws) {
</strong><strong>                    ws.send(`I'm not very smart, all I can do is repeat: "${event.data}"`);
</strong><strong>                }
</strong><strong>            };
</strong><strong>        })
</strong><strong>    );
</strong>    
    const server = serve({
        fetch: app.fetch,
        port
    });

<strong>    injectWebSocket(server);
</strong>
}
</code></pre>

Auth utilities:

[Source code](https://github.com/InseeFrLab/todo-rest-api/blob/e00a8a6ed95514c6be4b210506a22b0f0acf24a0/src/auth.ts#L95-L139)

{% code title="src/auth.ts" %}
```typescript
import { oidcSpa, extractRequestAuthContext } from "oidc-spa/server";
import { z } from "zod";
import { HTTPException } from "hono/http-exception";
import type { HonoRequest } from "hono";

const { bootstrapAuth, validateAndDecodeAccessToken } = oidcSpa
    .withExpectedDecodedAccessTokenShape({
        decodedAccessTokenSchema: z.object({
            sub: z.string(),
            name: z.string(),
            email: z.string().optional(),
            realm_access: z
                .object({
                    roles: z.array(z.string())
                })
                .optional()
        })
    })
    .createUtils();

export { bootstrapAuth };

export type User = {
    id: string;
    name: string;
    email: string | undefined;
};

export async function getUser(/* ... */): Promise<User> { /* ... */ }

export async function getUser_ws(params: { req: HonoRequest }) {
    const { req } = params;

    const value = req.header("Sec-WebSocket-Protocol");

    if (value === undefined) {
        throw new HTTPException(400); // Bad Request
    }

    const accessToken = value
        .split(",")
        .map(p => p.trim())
        .map(p => {
            const match = p.match(/^authorization_bearer_(.+)$/);

            if (match === null) {
                return undefined;
            }

            return match[1];
        })
        .filter(t => t !== undefined)[0];

    if (accessToken === undefined) {
        throw new HTTPException(400); // Bad Request
    }

    const { isSuccess, debugErrorMessage, decodedAccessToken } =
        await validateAndDecodeAccessToken({
            scheme: "Bearer",
            accessToken,
            // NOTE: WebSocket upgrades are out of scope for DPoP.
            // There's no RFC-defined way to send and validate a DPoP proof
            // on the WebSocket Upgrade request.
            // We accept the access token as bearer-like for the WS upgrade only.
            // If you need DPoP-grade guarantees on the socket, add an app-level handshake.
            rejectIfAccessTokenDPoPBound: false
        });

    if (!isSuccess) {
        console.warn(debugErrorMessage);
        throw new HTTPException(401); // Unauthorized
    }

    const { sub, name, email } = decodedAccessToken;

    const user: User = {
        id: sub,
        name,
        email
    };

    return user;
}
```
{% endcode %}

### Client-side code

[Source code](https://github.com/InseeFrLab/vite-insee-starter/blob/053da1b58e76a783aaa36dba1f371f2c46810c32/src/chat.ts#L28-L39)

<pre class="language-typescript" data-title=""><code class="lang-typescript">import { Evt, type StatefulReadonlyEvt } from "evt";
import { Deferred } from "evt/tools/Deferred";
import { getOidc } from "~/oidc";
import { assert } from "tsafe";

export type Chat = {
    evtMessages: StatefulReadonlyEvt&#x3C;Chat.Message[]>;
    sendMessage: (message: string) => void;
};

export namespace Chat {
    export type Message = {
        origin: "client" | "server";
        message: string;
    };
}

function createChat(): Chat {
    const evtMessages = Evt.create&#x3C;Chat.Message[]>([]);

    const dSocket = new Deferred&#x3C;WebSocket>();

    (async () => {
        const oidc = await getOidc();

        assert(oidc.isUserLoggedIn);

<strong>        const url = new URL(import.meta.env.VITE_TODOS_API_URL); // ex: https://api.my-company.com
</strong>
<strong>        url.protocol = url.protocol === "https:" ? "wss:" : "ws:";
</strong>
<strong>        url.pathname += "ws";
</strong>
<strong>        const socket = new WebSocket(
</strong><strong>            url.href, // ex: wss://api.my-company.com/ws
</strong><strong>            
</strong><strong>            // NOTE: This is a common workaround to the fact that the WebSocket API
</strong><strong>            // does not allow to set custom headers to the UPGRADE request.
</strong><strong>            // So we use the protocol and on the server read the Sec-WebSocket-Protocol header.
</strong><strong>            [`authorization_bearer_${await oidc.getAccessToken()}` ]
</strong><strong>        );
</strong>
        socket.addEventListener("message", event => {
            evtMessages.state = [
                ...evtMessages.state,
                {
                    origin: "server",
                    message: event.data
                }
            ];
        });

        socket.addEventListener("error", err => {
            console.error("socket error", err);
        });

        socket.addEventListener("open", ()=> {
            dSocket.resolve(socket);
        });
    })();

    return {
        evtMessages,
        sendMessage: async message => {
            evtMessages.state = [
                ...evtMessages.state,
                {
                    origin: "client",
                    message
                }
            ];
            const socket = await dSocket.pr;
            socket.send(message);
        }
    };
}

let chat: Chat | undefined = undefined;

export function getChat() {
    if (chat === undefined) {
        chat = createChat();
    }
    return chat;
}

</code></pre>

The source of the React component that consumes `getChat` is [here](https://github.com/InseeFrLab/vite-insee-starter/blob/053da1b58e76a783aaa36dba1f371f2c46810c32/src/routes/chat.tsx#L18-L83).
