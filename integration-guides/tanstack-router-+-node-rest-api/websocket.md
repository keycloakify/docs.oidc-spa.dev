---
icon: right-left-large
---

# WebSocket

WebSockets start as a normal HTTP request (`Upgrade: websocket`).\
That means you can validate the access token **once**, during the upgrade.

The main gotcha: browsers can’t set custom headers for `new WebSocket()`.\
So you usually can’t send `Authorization: Bearer ...` directly.

### Client → server: how to send the access token

#### Recommended (browser-friendly): `Sec-WebSocket-Protocol`

You can pass “subprotocols” from the browser.\
They end up in the `Sec-WebSocket-Protocol` header during the upgrade request.

Example idea:

* First protocol is a fixed marker: `oidc`
* Second protocol is the JWT access token

{% hint style="warning" %}
If you enabled DPoP on the frontend, WebSockets won’t carry per-request DPoP proofs.\
DPoP-bound access tokens may fail validation on the backend.
{% endhint %}

#### Fallback: query string

`wss://api.example.com/ws?access_token=...` works everywhere.\
But tokens in URLs tend to leak in logs and monitoring. Avoid if possible.

### Node.js + `ws` example

This example:

* extracts the token from `Sec-WebSocket-Protocol`
* maps it to `Authorization: Bearer ...` on the upgrade request
* uses `oidc-spa/server` to validate and decode the access token
* rejects the upgrade with `401/403` if validation fails

{% code title="src/main.ts" %}
```ts
import { createServer } from "node:http";
import { WebSocketServer } from "ws";
import type { IncomingMessage } from "node:http";
import { bootstrapAuth, getUserFromWsUpgrade } from "./auth"; // See below

function getAccessTokenFromSubprotocolHeader(req: IncomingMessage): string | undefined {
    const header = req.headers["sec-websocket-protocol"];

    if (typeof header !== "string") {
        return undefined;
    }

    // Example: "oidc, eyJhbGciOi..."
    const protocols = header
        .split(",")
        .map(s => s.trim())
        .filter(Boolean);

    // We expect ["oidc", "<access_token>"]
    if (protocols[0] !== "oidc") {
        return undefined;
    }

    return protocols[1];
}

function startWsServer() {

    bootstrapAuth({
        implementation: "real", // or "mock"
        issuerUri: process.env.OIDC_ISSUER_URI!,
        expectedAudience: process.env.OIDC_AUDIENCE
    });

    const server = createServer();

    const wss = new WebSocketServer({
        noServer: true,
        // Important: pick ONLY the marker protocol to avoid echoing the token back.
        handleProtocols(protocols) {
            return protocols.has("oidc") ? "oidc" : false;
        }
    });

    server.on("upgrade", async (req, socket, head) => {

        // Browser-friendly token transport: subprotocols.
        const accessToken = getAccessTokenFromSubprotocolHeader(req);

        if (accessToken) {
            req.headers.authorization = `Bearer ${accessToken}`;
        }

        const user = await getUserFromWsUpgrade({ req, socket });

        wss.handleUpgrade(req, socket, head, ws => {
            // Attach user to the connection (simple example).
            (ws as any).user = user;
            wss.emit("connection", ws, req);
        });
    });

    wss.on("connection", (ws) => {
        const user = (ws as any).user as { id: string };

        ws.send(JSON.stringify({ type: "welcome", userId: user.id }));

        ws.on("message", raw => {
            // Your app messages go here.
            // You already authenticated the connection at upgrade time.
            ws.send(raw.toString());
        });
    });

    server.listen(parseInt(process.env.PORT ?? "3000"), () => {
        console.log("WebSocket server listening");
    });
}
```
{% endcode %}

### Auth utilities

{% code title="src/auth.ts" %}
```ts
import { oidcSpa, extractRequestAuthContext } from "oidc-spa/server";
import { z } from "zod";
import type { IncomingMessage } from "node:http";
import type { Socket } from "node:net";

const { bootstrapAuth, validateAndDecodeAccessToken } = oidcSpa
    .withExpectedDecodedAccessTokenShape({
        decodedAccessTokenSchema: z.object({
            sub: z.string(),
            // Keycloak specific, convention to manage authorization.
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
};

function rejectUpgrade(params: {
    socket: Socket;
    statusCode: 400 | 401 | 403;
}): never {
    const { socket, statusCode } = params;

    // Minimal HTTP response. Good enough for WebSocket upgrade failures.
    socket.write(`HTTP/1.1 ${statusCode}\r\n\r\n`);
    socket.destroy();

    return new Promise<never>(() => {});
}

export async function getUserFromWsUpgrade(params: {
    req: IncomingMessage;
    socket: Socket;
    requiredRole?: "realm-admin" | "support-staff";
}): Promise<User | never> {
    const { req, socket, requiredRole } = params;

    const requestAuthContext = extractRequestAuthContext({
        request: req,
        // Set this to false only if you don't have a reverse HTTP proxy in front of your
        // server. (Almost never the case in modern deployments).
        trustProxy: true
    });

    if (!requestAuthContext) {
        console.warn("Anonymous WebSocket upgrade");
        return rejectUpgrade({ socket, statusCode: 401 });
    }

    if (!requestAuthContext.isWellFormed) {
        console.warn(requestAuthContext.debugErrorMessage);
        return rejectUpgrade({ socket, statusCode: 400 });
    }

    const { isSuccess, debugErrorMessage, decodedAccessToken } =
        await validateAndDecodeAccessToken(requestAuthContext.accessTokenAndMetadata);

    if (!isSuccess) {
        console.warn(debugErrorMessage);
        return rejectUpgrade({ socket, statusCode: 401 });
    }

    if (requiredRole) {
        if (!decodedAccessToken.realm_access?.roles.includes(requiredRole)) {
            console.warn(`User missing role: ${requiredRole}`);
            return rejectUpgrade({ socket, statusCode: 403 });
        }
    }

    return { id: decodedAccessToken.sub };
}
```
{% endcode %}

### Notes for production

* Validate `Origin` during the upgrade if you rely on browser clients.\
  WebSockets are not protected by CORS.
* Decide what happens when tokens expire.\
  Common approach: close the socket and let the client reconnect.
