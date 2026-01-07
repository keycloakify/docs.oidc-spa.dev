---
icon: paw-claws
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/oygeayjvIPxroUcp3jt4/integration-guides/backend-token-validation/nestjs
---

# NestJS

{% hint style="info" %}
If you prefer a more "Nestish" experience, there's a comunity wrapper around oidc-spa/server:

[https://github.com/mwolf1989/nestjs-spa-oidc](https://github.com/mwolf1989/nestjs-spa-oidc)
{% endhint %}

This is how your Nest API would typically look like.

<pre class="language-ts" data-title="src/main.ts"><code class="lang-ts">import { NestFactory } from "@nestjs/core";
import { AppModule } from "./app.module";
import { bootstrapAuth } from "./auth"; // See below
+import { ConfigService } from "@nestjs/config";

async function bootstrap() {
    const app = await NestFactory.create(AppModule, /* Any adapter */);

    // Requires ConfigModule.forRoot() somewhere in your imports (typically AppModule).
    const configService = app.get(ConfigService);

<strong>    bootstrapAuth({
</strong><strong>        implementation: "real", // or "mock", see: https://docs.oidc-spa.dev/v/v8/integration-guides/backend-token-validation/mock-modes
</strong><strong>        issuerUri: configService.get("OIDC_ISSUER_URI")!,
</strong><strong>        expectedAudience: configService.get("OIDC_AUDIENCE")
</strong><strong>    });
</strong>

    await app.listen(parseInt(configService.get("PORT") ?? "3000"));
}

bootstrap();
</code></pre>

And this is how your controlled would look:

<pre class="language-ts" data-title="src/todos.controller.ts"><code class="lang-ts">import * as fs from "node:fs/promises";
import { Controller, Get, Param, Req } from "@nestjs/common";
<strong>import { getUser } from "./auth";
</strong>
@Controller("api")
export class TodosController {
    @Get("todos")
    async getTodos(@Req() req) {
<strong>        const user = await getUser({ req });
</strong>        const json = await fs.readFile(`todos_${user.id}.json`, "utf8");
        return JSON.parse(json);
    }

    @Get("todos-for-support/:userId")
    async getTodosForSupportStaff(
        @Req() req, 
        @Param("userId") userId: string
    ) {
<strong>        // Will reject the request if user making the request
</strong><strong>        // doesn't have "support-staff" role.
</strong><strong>        await getUser({ req, requiredRole: "support-staff" });
</strong>        const json = await fs.readFile(`todos_${userId}.json`, "utf8");
        return JSON.parse(json);
    }
}
</code></pre>

This is the only “integration” code you need:

{% code title="src/auth.ts" %}
```ts
import { BadRequestException, ForbiddenException, UnauthorizedException } from "@nestjs/common";
import { oidcSpa, extractRequestAuthContext, type AnyRequest } from "oidc-spa/server";
import { z } from "zod";

const { bootstrapAuth, validateAndDecodeAccessToken } = oidcSpa
    .withExpectedDecodedAccessTokenShape({
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

// Your local respresentation of a user
export type User = {
    id: string;
    name: string;
    email: string | undefined;
};

export async function getUser(params: {
    // This can be an Express Request object, a FastifyRequest object
    // or really any well know object that represent a request,
    // oidc-spa will normalize the representation internally.
    // so this function will work regardless of the HTTP framework
    // you're using to bootstrap your NestJS app.
    req: AnyRequest;
    requiredRole?: "realm-admin" | "support-staff";
}): Promise<User> {

    const { req, requiredRole } = params;

    const requestAuthContext = extractRequestAuthContext({
        request: req,
        // Set this to false only if you don't have a reverse HTTP proxy in front of your
        // server. (Almost never the case in modern deployments).
        trustProxy: true
    });

    if (!requestAuthContext) {
        console.warn("Anonymous request");
        throw new UnauthorizedException();
    }

    if (!requestAuthContext.isWellFormed) {
        console.warn(requestAuthContext.debugErrorMessage);
        throw new BadRequestException();
    }

    const { isSuccess, debugErrorMessage, decodedAccessToken } =
        await validateAndDecodeAccessToken(
            requestAuthContext.accessTokenAndMetadata
        );

    if (!isSuccess) {
        console.warn(debugErrorMessage);
        throw new UnauthorizedException();
    }

    if (requiredRole) {
        if (!decodedAccessToken.realm_access?.roles.includes(requiredRole)) {
            console.warn(`User missing role: ${requiredRole}`);
            throw new ForbiddenException();
        }
    }

    const { sub, name, email } = decodedAccessToken;

    return { id: sub, name, email };
}
```
{% endcode %}
