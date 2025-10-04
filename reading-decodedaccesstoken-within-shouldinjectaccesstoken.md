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
  // see: https://docs.oidc-spa.dev/release-notes/reading-decodedaccesstoken-within-shouldinjectaccesstoken
<strong>  override allowDecodedIdTokenAccessInShouldInjectAccessToken = true;
</strong>}
</code></pre>

Then you must make sure that every requests that depend on this rule are delayed after oidc.prInitialized has resolved, like for example by doing:

```typescript
@Injectable({ providedIn: 'root' })
export class TodoService {
  private readonly http = inject(HttpClient);
  private readonly apiUrl = TODO_API_URL;
  private readonly oidc = inject(Oidc);

  getPublicAndAdminTodos(): Observable<Todo[]> {
    return from(this.oidc.prInitialized).pipe(
      switchMap(() =>
        this.http.get<Todo[]>(`${this.apiUrl}/todos`, {
          params: { _limit: 5 },
          context: new HttpContext().set(INCLUDE_ACCESS_TOKEN_IF_ADMIN, true),
        })
      )
    );
  }
```

Sorry for the inconvegnience.
