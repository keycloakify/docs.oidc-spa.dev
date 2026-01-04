---
icon: cubes
metaLinks:
  alternates:
    - /broken/spaces/UhNOMoIddws1XoAnT5Nn/pages/jPYFd1DOgIwEr8YEhJ6x
---

# tRPC

This is how your API handler would typically look like:

<pre class="language-ts" data-title="src/main.ts"><code class="lang-ts">import express from "express";
import * as fs from "node:fs/promises";
import { z } from "zod";
import { initTRPC } from "@trpc/server";
import { createExpressMiddleware } from "@trpc/server/adapters/express";
import type { Request } from "express";
import { bootstrapAuth, getUser } from "./auth"; // See below

function startExpressTrpcServer() {
<strong>    bootstrapAuth({
</strong><strong>        implementation: "real", // or "mock", see: https://docs.oidc-spa.dev/v/v8/integration-guides/backend-token-validation/mock-modes
</strong><strong>        issuerUri: process.env.OIDC_ISSUER_URI!,
</strong><strong>        expectedAudience: process.env.OIDC_AUDIENCE
</strong><strong>    });
</strong>
    const app = express();

<strong>    /**
</strong><strong>     * Key idea: Wether you use use Express or something else
</strong><strong>     * as underlying HTTP framework, just expose whatever the 
</strong><strong>     * request object representation is to the global context.
</strong><strong>     * oidc-spa will be able to extract the auth context from it.
</strong><strong>     */
</strong><strong>    const createContext = ({ req }: { req: Request }) => ({ req });
</strong>
    type Context = ReturnType&#x3C;typeof createContext>;

    const t = initTRPC.context&#x3C;Context>().create();

    const appRouter = t.router({
        todos: t.procedure.query(async ({ ctx }) => {
<strong>            const user = await getUser({ req: ctx.req });
</strong>            const json = await fs.readFile(
<strong>                `todos_${user.id}.json`, 
</strong>                "utf8"
            );
            return JSON.parse(json);
        }),

        todosForSupportStaff: t.procedure
            .input(z.object({ userId: z.string() }))
            .query(async ({ ctx, input }) => {
                // Will reject the request if user making the request
                // doesn't have "support-staff" role
<strong>                await getUser({ req: ctx.req, requiredRole: "support-staff" });
</strong>                const json = await fs.readFile(`todos_${input.userId}.json`, "utf8");
                return JSON.parse(json);
            })
    });

    app.use(
        "/trpc",
        // Needed for tRPC POST requests.
        express.json(),
        createExpressMiddleware({
            router: appRouter,
            createContext
        })
    );

    app.listen(parseInt(process.env.PORT ?? "3000"), () => {
        console.log("Server running");
    });
}
</code></pre>

Let's see how to export the utils to make it happen:

{% code title="src/auth.ts" %}
```ts
import { oidcSpa, extractRequestAuthContext } from "oidc-spa/server";
import { z } from "zod";
import { TRPCError } from "@trpc/server";
import type { Request } from "express";

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
}): Promise<User> {

    const { req, requiredRole } = params;

    const requestAuthContext = extractRequestAuthContext({
        // Here request accept any common representation of a request
        // Request | IncomingMessage | HonoRequest | FastifyRequest ...
        request: req,
        // Set this to false only if you don't have a reverse HTTP proxy in front of your
        // server. (Almost never the case in modern deployments).
        trustProxy: true
    });

    if (!requestAuthContext) {
        // Demo shortcut: we throw on missing Authorization, but a mixed
        // public/private procedure could instead return undefined here and let
        // the caller decide whether to process an anonymous request.
        console.warn("Anonymous request");
        throw new TRPCError({ code: "UNAUTHORIZED" });
    }

    if (!requestAuthContext.isWellFormed) {
        console.warn(requestAuthContext.debugErrorMessage);
        throw new TRPCError({ code: "BAD_REQUEST" });
    }

    const { isSuccess, debugErrorMessage, decodedAccessToken } =
        await validateAndDecodeAccessToken(
            requestAuthContext.accessTokenAndMetadata
        );

    if (!isSuccess) {
        console.warn(debugErrorMessage);
        throw new TRPCError({ code: "UNAUTHORIZED" });
    }

    // Your custom Authorization logic: Grant per request access depending
    // on the access token claim.
    if (requiredRole) {
        if (!decodedAccessToken.realm_access?.roles.includes(requiredRole)) {
            console.warn(`User missing role: ${requiredRole}`);
            throw new TRPCError({ code: "FORBIDDEN" });
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
