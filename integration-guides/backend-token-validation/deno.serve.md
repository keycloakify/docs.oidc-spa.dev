---
icon: dinosaur
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/oygeayjvIPxroUcp3jt4/integration-guides/backend-token-validation/deno.serve
---

# Deno.serve

This is how your API handler would typically look like:

<pre class="language-ts" data-title="src/main.ts"><code class="lang-ts"><strong>import { bootstrapAuth, getUser } from "./auth.ts"; // See below
</strong>
bootstrapAuth({
    implementation: "real", // or "mock", see: https://docs.oidc-spa.dev/v/v8/integration-guides/backend-token-validation/mock-modes
    issuerUri: Deno.env.get("OIDC_ISSUER_URI")!,
    expectedAudience: Deno.env.get("OIDC_AUDIENCE") ?? undefined
});

Deno.serve(async (request: Request) => {
    const url = new URL(request.url);

    if (request.method === "GET" &#x26;&#x26; url.pathname === "/api/todos") {

<strong>        const user = await getUser({ req: request });
</strong>
<strong>        // We got an exception, validation failed
</strong><strong>        if (user instanceof Response) {
</strong><strong>            const response = user;
</strong><strong>            return response;
</strong><strong>        }
</strong>
        const json = await Deno.readTextFile(
<strong>            `todos_${user.id}.json`
</strong>        );

        return new Response(json, {
            status: 200,
            headers: { "content-type": "application/json" }
        });

    }

    /**
     * Support staff endpoint.
     * Example: GET /api/todos/1234
     */
    if (request.method === "GET" &#x26;&#x26; url.pathname.startsWith("/api/todos/")) {
        let userId: string;

        try {
            userId = decodeURIComponent(url.pathname.replace("/api/todos/", ""));
        } catch {
            return new Response("bad request", { status: 400 });
        }

        if (!userId || userId.includes("/")) {
            return new Response("bad request", { status: 400 });
        }
        
        {

<strong>            // Will reject the request if user making the request
</strong><strong>            // doesn't have "support-staff" role
</strong><strong>            const user = await getUser({ req: request, requiredRole: "support-staff" });
</strong>    
<strong>            // We got an exception, validation failed
</strong><strong>            if (user instanceof Response) {
</strong><strong>                const response = user;
</strong><strong>                return response;
</strong><strong>            }
</strong>        
        }

<strong>        const json = await Deno.readTextFile(`todos_${userId}.json`);
</strong>
        return new Response(json, {
            status: 200,
            headers: { "content-type": "application/json" }
        });
    }

    return new Response("not found", { status: 404 });
});
</code></pre>

Let's see how to export the utils to make it happen:

{% code title="src/auth.ts" %}
```ts
import { oidcSpa, extractRequestAuthContext } from "npm:oidc-spa@latest/server";
import { z } from "npm:zod@latest";

const { bootstrapAuth, validateAndDecodeAccessToken } = oidcSpa
    .withExpectedDecodedAccessTokenShape({
        // This is purely declarative. Here you'll specify
        // the claim that you expect to be present in the access token payload.
        decodedAccessTokenSchema: z.object({
            sub: z.string(),
            name: z.string(),
            email: z.string().optional(),
            // Keycloak specific, convention to manage authorization.
            realm_access: z.object({
                roles: z.array(z.string())
            }).optional()
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
        // Set this to false only if you don't have a reverse HTTP proxy in front of your
        // server. (Almost never the case in modern deployments).
        trustProxy: true
    });

    if (!requestAuthContext) {
        // Demo shortcut: we return 401 on missing Authorization, but a mixed
        // public/private endpoint could instead return undefined here and let
        // the caller decide whether to process an anonymous request.
        console.warn("Anonymous request");
        return new Response("unauthorized", { status: 401 });
    }

    if (!requestAuthContext.isWellFormed) {
        console.warn(requestAuthContext.debugErrorMessage);
        return new Response("bad request", { status: 400 });
    }

    const { isSuccess, debugErrorMessage, decodedAccessToken } =
        await validateAndDecodeAccessToken(
            requestAuthContext.accessTokenAndMetadata
        );

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

    // Here you can potentially enrich the user object with additional
    // data that you would retrieve from your database if the access token
    // claim does not contain everything you need.

    const { sub, name, email } = decodedAccessToken;

    const user: User = { id: sub, name, email };

    return user;
}
```
{% endcode %}
