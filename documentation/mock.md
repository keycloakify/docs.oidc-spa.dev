# 🎭 Mock

For certain use cases, you may want a mock adapter to simulate user authentication without involving an actual authentication server.

This approach is useful when building an app where user authentication is a feature but not a requirement. It also proves beneficial for running tests or in Storybook environments.

{% embed url="https://youtu.be/9agS61jJjqg" %}

{% tabs %}
{% tab title="Vanilla API" %}
<pre class="language-typescript"><code class="lang-typescript">import { createOidc } from "oidc-spa";
<strong>import { createMockOidc } from "oidc-spa/mock";
</strong>import { z } from "zod";

const decodedIdTokenSchema = z.object({
    sub: z.string(),
    preferred_username: z.string()
});

const publicUrl= import.meta.env.BASE_URL;

const oidc = !import.meta.env.VITE_OIDC_ISSUER
<strong>    ? await createMockOidc({
</strong><strong>          isUserInitiallyLoggedIn: false,
</strong><strong>          publicUrl,
</strong><strong>          mockedTokens: {
</strong><strong>              decodedIdToken: {
</strong><strong>                  sub: "123",
</strong><strong>                  preferred_username: "john doe"
</strong><strong>              } satisfies z.infer&#x3C;typeof decodedIdTokenSchema>
</strong><strong>          }
</strong><strong>      })
</strong>    : await createOidc({
          issuerUri: import.meta.env.VITE_OIDC_ISSUER,
          clientId: import.meta.env.VITE_OIDC_CLIENT_ID,
          publicUrl
      });
</code></pre>
{% endtab %}

{% tab title="React API" %}
<pre class="language-typescript" data-title="src/oidc.ts"><code class="lang-typescript">import { createReactOidc } from "oidc-spa/react";
<strong>import { createMockReactOidc } from "oidc-spa/mock/react";
</strong>import { z } from "zod";

const decodedIdTokenSchema = z.object({
    sub: z.string(),
    preferred_username: z.string()
});

const publicUrl = import.meta.env.BASE_URL;

export const { OidcProvider, useOidc, prOidc } =
    !import.meta.env.VITE_OIDC_ISSUER ?
<strong>        createMockReactOidc({
</strong><strong>            isUserInitiallyLoggedIn: false,
</strong><strong>            publicUrl,
</strong><strong>            mockedTokens: {
</strong><strong>                decodedIdToken: {
</strong><strong>                    sub: "123",
</strong><strong>                    preferred_username: "john doe"
</strong><strong>                } satisfies z.infer&#x3C;typeof decodedIdTokenSchema>
</strong><strong>            }
</strong><strong>        }) :
</strong>        createReactOidc({
            issuerUri: import.meta.env.VITE_OIDC_ISSUER,
            clientId: import.meta.env.VITE_OIDC_CLIENT_ID,
            publicUrl,
            decodedIdTokenSchema
        });
</code></pre>
{% endtab %}
{% endtabs %}
