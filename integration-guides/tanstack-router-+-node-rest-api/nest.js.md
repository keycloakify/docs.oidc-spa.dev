---
icon: paw-claws
---

# Nest.js

{% hint style="info" %}
If you prefer a more "Nestish" experience, there's a comunity wrapper around oidc-spa/server: &#x20;

[https://github.com/mwolf1989/nestjs-spa-oidc](https://github.com/mwolf1989/nestjs-spa-oidc)
{% endhint %}

This is how your Nest API would typically look like.

<pre class="language-ts" data-title="src/main.ts"><code class="lang-ts">import { NestFactory } from "@nestjs/core";
import { AppModule } from "./app.module";
import { bootstrapAuth } from "./auth"; // See below

async function bootstrap() {
<strong>    bootstrapAuth({
</strong><strong>        implementation: "real", // or "mock"
</strong><strong>        issuerUri: process.env.OIDC_ISSUER_URI!,
</strong><strong>        expectedAudience: process.env.OIDC_AUDIENCE
</strong><strong>    });
</strong>
    const app = await NestFactory.create(AppModule, /* Any adapter */);

    await app.listen(parseInt(process.env.PORT ?? "3000"));
}

bootstrap();
</code></pre>

Now, in your controllers, keep using the raw request object.

{% code title="src/todos.controller.ts" %}
```ts
import * as fs from "node:fs/promises";
import { Controller, Get, Param, Req } from "@nestjs/common";
import { getUser } from "./auth";

@Controller("api")
export class TodosController {
    @Get("todos")
    async getTodos(@Req() req) {
        const user = await getUser(req);
        const json = await fs.readFile(`todos_${user.id}.json`, "utf8");
        return JSON.parse(json);
    }

    @Get("todos-for-support/:userId")
    async getTodosForSupportStaff(
        @Req() req, 
        @Param("userId") userId: string
    ) {
        // Will reject the request if user making the request
        // doesn't have "support-staff" role.
        await getUser(req, "support-staff");
        const json = await fs.readFile(`todos_${userId}.json`, "utf8");
        return JSON.parse(json);
    }
}
```
{% endcode %}

This is the only “integration” code you need: &#x20;

{% code title="src/auth.ts" %}
```ts
import { BadRequestException, ForbiddenException, UnauthorizedException } from "@nestjs/common";
import { oidcSpa, extractRequestAuthContext, type AnyRequest } from "oidc-spa/server";
import { z } from "zod";

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

export async function getUser(
    // This can be an Express Request object, a FastifyRequest object
    // or really any well know object that represent a request,
    // oidc-spa will normalize the representation internally.
    // so this function will work regardless of the HTTP framework
    // you're using to bootstrap your NestJS app.
    req: AnyRequest,
    requiredRole?: "realm-admin" | "support-staff"
): Promise<User> {
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

    return { id: decodedAccessToken.sub };
}
```
{% endcode %}
