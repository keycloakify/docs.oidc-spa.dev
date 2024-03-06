# 🔁 Tokens renewal

You probably don't need to do it. The token refresh is handled automatically for you, however you can manually trigger a token refresh: &#x20;

{% tabs %}
{% tab title="Vanilla API" %}
```typescript
import { createOidc } from "oidc-spa";

const oidc = await createOidc({ ... });

if( oidc.isUserLoggedIn ){
   oidc.renewToken();
}
```
{% endtab %}

{% tab title="React API" %}
```tsx
import { useOidc } from "oidc";
import { useEffect } from "react";

function MyComponent(){

  const { renewTokens } = useOidc({ assertUserLoggedIn: true });
  
  useEffect(()=> {
      renewTokens();
  }, []);
  
  return <>...</>;

}
```
{% endtab %}
{% endtabs %}
