---
icon: dinosaur
---

# Deno.serve

This is how your API handler would typically look like:

<pre class="language-ts" data-title="src/main.ts"><code class="lang-ts"><strong>import { bootstrapAuth, getUser } from "./auth.ts"; // See below
</strong>
bootstrapAuth({
    implementation: "real", // or "mock"
    issuerUri: Deno.env.get("OIDC_ISSUER_URI")!,
    expectedAudience: Deno.env.get("OIDC_AUDIENCE") ?? undefined
});

Deno.serve(async (request: Request) => {
    const url = new URL(request.url);

    if (request.method === "GET" &#x26;&#x26; url.pathname === "/api/todos") {

<strong>        const user = await getUser(request);
</strong>
<strong>        // We got an exception, validation failed
</strong><strong>        if (user instanceof Response) {
</strong><strong>            return userOrResponse;
</strong><strong>        }
</strong>
<strong>        const json = await Deno.readTextFile(`todos_${user.id}.json`);
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
};

export async function getUser(
    request: Request,
    requiredRole?: "realm-admin" | "support-staff"
): Promise<User | Response> {
    const requestAuthContext = extractRequestAuthContext({
        request,
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

    const user: User = {
        id: decodedAccessToken.sub
    };

    return user;
}
```
{% endcode %}
