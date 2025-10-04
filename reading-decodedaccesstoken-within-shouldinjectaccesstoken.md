---
hidden: true
icon: angular
---

# Reading decodedAccessToken within shouldInjectAccessToken()

If you are seeing this page this means that you are in a very edge case where you have providerAwaitsInitialization set to false: &#x20;

<pre class="language-typescript"><code class="lang-typescript">@Injectable({ providedIn: 'root' })
export class Oidc extends AbstractOidcService&#x3C;DecodedIdToken> {
<strong>  override providerAwaitsInitialization = false;
</strong>}
</code></pre>

And you are reading the decodedIdToken to decide if the the access token should be used as bearer for a given request, like for example by doing:&#x20;

```typescript
Oidc.createBearerInterceptor({
  shouldInjectAccessToken: (req) => {
    const oidc = inject(Oidc);

    if (req.context.get(INCLUDE_ACCESS_TOKEN_IF_ADMIN)) {
      return oidc.isUserLoggedIn && oidc.$decodedIdToken().realm_access?.roles.includes("admin");
    }

    return false;
  },
})
```

It's fine to do that but due to a technical detail in the API design we can't guarantiy you that the decision will always resolve as it should if the request is made BEFORE oidc.prInitialized has resolved. &#x20;

As a result you must do two things, the first one is to declare that you have read this message and understand the implication. &#x20;

<pre class="language-typescript"><code class="lang-typescript">@Injectable({ providedIn: 'root' })
export class Oidc extends AbstractOidcService&#x3C;DecodedIdToken> {
  override providerAwaitsInitialization = false;
<strong>  override allowDecodedIdTokenAccessInShouldInjectAccessToken = true;
</strong>}
</code></pre>

