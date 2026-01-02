---
icon: cloudflare
metaLinks:
  alternates:
    - /broken/spaces/UhNOMoIddws1XoAnT5Nn/pages/7iLCD0QZbmGywDt5qJd7
---

# Cloudflare Workers

This is how your API handler would typically look like:

<pre class="language-ts" data-title="src/worker.ts"><code class="lang-ts"><strong>import { bootstrapAuth, getUser } from "./auth"; // See below
</strong>
type Env = {
    OIDC_ISSUER_URI: string;
    OIDC_AUDIENCE?: string;
};

let isBootstrapped = false;

function ensureBootstrapped(env: Env) {
    if (isBootstrapped) {
        return;
    }

<strong>    bootstrapAuth({
</strong><strong>        implementation: "real", // or "mock", see: https://docs.oidc-spa.dev/v/v8/integration-guides/backend-token-validation/mock-modes
</strong><strong>        issuerUri: env.OIDC_ISSUER_URI,
</strong><strong>        expectedAudience: env.OIDC_AUDIENCE ?? undefined
</strong><strong>    });
</strong>
    isBootstrapped = true;
}

export default {
    async fetch(request: Request, env: Env): Promise&#x3C;Response> {
        ensureBootstrapped(env);

        const url = new URL(request.url);

        if (request.method === "GET" &#x26;&#x26; url.pathname === "/api/todos") {

<strong>            const user = await getUser({ req: request });
</strong>
<strong>            // We got a Response, validation failed
</strong><strong>            if (user instanceof Response) {
</strong><strong>                return user;
</strong><strong>            }
</strong>
            // Replace this with KV / D1 / R2 / your DB call.
            const json = JSON.stringify([
                { id: "1", label: "Write documentation", ownerId: user.id }
            ]);

            return new Response(json, {
                status: 200,
                headers: { "content-type": "application/json" }
            });
        }

        /**
         * Support staff endpoint.
         * Example: GET /api/todos-for-support/1234
         */
        if (
            request.method === "GET" &#x26;&#x26;
            url.pathname.startsWith("/api/todos-for-support/")
        ) {
            let userId: string;

            try {
                userId = decodeURIComponent(
                    url.pathname.replace("/api/todos-for-support/", "")
                );
            } catch {
                return new Response("bad request", { status: 400 });
            }

            if (!userId || userId.includes("/")) {
                return new Response("bad request", { status: 400 });
            }

            {
<strong>                // Will reject the request if user making the request
</strong><strong>                // doesn't have "support-staff" role
</strong><strong>                const user = await getUser({
</strong><strong>                    req: request,
</strong><strong>                    requiredRole: "support-staff"
</strong><strong>                });
</strong>
<strong>                if (user instanceof Response) {
</strong><strong>                    return user;
</strong><strong>                }
</strong>            }

            // Replace this with KV / D1 / R2 / your DB call.
            const json = JSON.stringify([
                { id: "1", label: "Support view", ownerId: userId }
            ]);

            return new Response(json, {
                status: 200,
                headers: { "content-type": "application/json" }
            });
        }

        return new Response("not found", { status: 404 });
    }
};
</code></pre>

### Auth utilities

Let’s see how to export the utils to make it happen:

{% code title="src/auth.ts" %}
```ts
import { oidcSpa, extractRequestAuthContext } from "oidc-spa/server";
import { z } from "zod";

const { bootstrapAuth, validateAndDecodeAccessToken } = oidcSpa
    .withExpectedDecodedAccessTokenShape({
        // This is purely declarative. Here you'll specify
        // the claim that you expect to be present in the access token payload.
        decodedAccessTokenSchema: z.object({
            sub: z.string(),
            name: z.string(),
            email: z.string().optional(),
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

// Your local representation of a user.
export type User = {
    id: string;
    name: string;
    email: string | undefined;
};

export async function getUser(params: {
    req: Request;
    requiredRole?: "realm-admin" | "support-staff";
}): Promise<User | Response> {

    const { req, requiredRole } = params;

    const requestAuthContext = extractRequestAuthContext({
        request: req,
        // Cloudflare Workers are always behind a reverse proxy.
        // This affects things like the computed request origin.
        trustProxy: true
    });

    if (!requestAuthContext) {
        console.warn("Anonymous request");
        return new Response("unauthorized", { status: 401 });
    }

    if (!requestAuthContext.isWellFormed) {
        console.warn(requestAuthContext.debugErrorMessage);
        return new Response("bad request", { status: 400 });
    }

    const { isSuccess, debugErrorMessage, decodedAccessToken } =
        await validateAndDecodeAccessToken(requestAuthContext.accessTokenAndMetadata);

    if (!isSuccess) {
        console.warn(debugErrorMessage);
        return new Response("unauthorized", { status: 401 });
    }

    // Your custom Authorization logic: Grant per request access depending
    // on the access token claim.
    if (requiredRole) {
        if (!decodedAccessToken.realm_access?.roles.includes(requiredRole)) {
            console.warn(`User missing role: ${requiredRole}`);
            return new Response("forbidden", { status: 403 });
        }
    }

    const { sub, name, email } = decodedAccessToken;

    const user: User = { id: sub, name, email };

    return user;
}
```
{% endcode %}
