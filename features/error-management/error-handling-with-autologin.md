# Error Handling - With AutoLogin

{% hint style="info" %}
This guide only applies if you have enabled [Auto Login](../auto-login.md).\
If you do **not** have Auto Login enabled, follow [this guide instead](error-handling-no-autologin.md).
{% endhint %}

Here is how you can gracefully handle oidc initialization errors:

{% tabs %}
{% tab title="Framework Agnostic" %}
```typescript
import { createOidc, type OidcInitializationError } from "oidc-spa/core";

const oidc = await createOidc({
    // ...
    autoLogin: true
})
// In autoLogin: false, createOidc never throws.
// In autoLogin: true, it can throw — but only OidcInitializationError —
// so you can safely narrow/cast here.
.catch(error => error as OidcInitializationError);

if( oidc instanceof Error ){

    const oidcInitializationError = oidc;
    
    // Use this to distinguish a misconfiguration from a temporary auth-server outage.
    // NOTE: below references should use `oidcInitializationError`.
    console.log(initializationError.isAuthServerLikelyDown);
    
    // Developer-only diagnostic with likely cause and fix.
    // Do not display this to end users.
    console.log(initializationError.message);
    
    alert("Our auth is down, sorry :(");
    
    // Halt the app in a typed-safe way (nothing renders until you decide otherwise).
    await Promise<never>(()=>{});
}
```
{% endtab %}

{% tab title="TanStack Start" %}
<pre class="language-tsx" data-title="src/routes/__root.tsx"><code class="lang-tsx">import { HeadContent, Scripts, createRootRoute } from "@tanstack/react-router";

import Header from "@/components/Header";
import { AutoLogoutWarningOverlay } from "@/components/AutoLogoutWarningOverlay";
<strong>import { useOidc } from "@/oidc";
</strong><strong>import type { OidcInitializationError } from "oidc-spa/core";
</strong>
export const Route = createRootRoute({
    // ...
    shellComponent: RootDocument
});

function RootDocument({ children }: { children: React.ReactNode }) {

<strong>    const { oidcInitializationError } = useOidc();
</strong>
    return (
        &#x3C;html lang="en">
            &#x3C;head>
                &#x3C;HeadContent />
            &#x3C;/head>
            &#x3C;body>
                &#x3C;div className="min-h-screen flex flex-col">
                    &#x3C;Header />
                    &#x3C;main className="flex flex-1 flex-col">
<strong>                        {oidcInitializationError ? (
</strong><strong>                            &#x3C;OidcErrorComponent oidcInitializationError={oidcInitializationError} />
</strong><strong>                        ) : (
</strong>                            children
<strong>                        )}
</strong>                    &#x3C;/main>
                &#x3C;/div>
                &#x3C;AutoLogoutWarningOverlay />
                &#x3C;Scripts />
            &#x3C;/body>
        &#x3C;/html>
    );
}

<strong>function OidcErrorComponent(props: { 
</strong><strong>    oidcInitializationError: OidcInitializationError;
</strong><strong>}){
</strong><strong>    const { oidcInitializationError } = props;
</strong><strong>    
</strong><strong>    // Distinguish misconfiguration vs. temporary auth-server outage.
</strong><strong>    console.log(oidcInitializationError.isAuthServerLikelyDown);
</strong>
<strong>    // Developer-only diagnostic with likely cause and fix.
</strong><strong>    // Do not display this to end users.
</strong><strong>    console.log(oidcInitializationError.message);
</strong>
<strong>    return &#x3C;h1>Our auth is down, sorry&#x3C;/h1>;
</strong><strong>    
</strong><strong>}
</strong>
</code></pre>
{% endtab %}

{% tab title="React SPAs" %}
<pre class="language-typescript" data-title="src/oidc.ts"><code class="lang-typescript">import { oidcSpa } from "oidc-spa/react-spa";

export const {
    bootstrapOidc,
    useOidc,
    getOidc,
    OidcInitializationGate,
<strong>    OidcInitializationErrorGate
</strong>} = oidcSpa
    .withExpectedDecodedIdTokenShape({ /* ... */ }),
    .withAutoLogin()
    .createUtils();
</code></pre>

<pre class="language-tsx"><code class="lang-tsx">import React from "react";
import ReactDOM from "react-dom/client";
import { App } from "./App";
import { 
    OidcInitializationGate, 
<strong>    OidcInitializationErrorGate 
</strong>} from "~/oidc";
import type { OidcInitializationError } from "oidc-spa/core";

ReactDOM.createRoot(document.getElementById("root")!).render(
    &#x3C;React.StrictMode>
        &#x3C;OidcInitializationGate>
<strong>            &#x3C;OidcInitializationErrorGate errorComponent={OidcErrorComponent} >
</strong>                &#x3C;App />
<strong>            &#x3C;/OidcInitializationErrorGate>
</strong>        &#x3C;/OidcInitializationGate>
    &#x3C;/React.StrictMode>
);

<strong>function OidcErrorComponent(props: { 
</strong><strong>    oidcInitializationError: OidcInitializationError;
</strong><strong>}){
</strong><strong>    const { oidcInitializationError } = props;
</strong><strong>    
</strong><strong>    // Distinguish misconfiguration vs. temporary auth-server outage.
</strong><strong>    console.log(oidcInitializationError.isAuthServerLikelyDown);
</strong>
<strong>    // Developer-only diagnostic with likely cause and fix.
</strong><strong>    // Do not display this to end users.
</strong><strong>    console.log(oidcInitializationError.message);
</strong>
<strong>    return &#x3C;h1>Our auth is down, sorry&#x3C;/h1>;
</strong><strong>    
</strong><strong>}
</strong></code></pre>
{% endtab %}

{% tab title="Angular" %}
{% code title="src/app/app.html" %}
```html
@if (oidc.initializationError) {
<h1>Our Auth is down, sorry :(</h1>
}@else{
<!-- Your app -->
}
```
{% endcode %}

<pre class="language-typescript"><code class="lang-typescript">@Component({
  selector: 'app-root',
  imports: [RouterOutlet, RouterLink, RouterLinkActive],
  templateUrl: './app.html',
})
export class App {
  oidc = inject(Oidc);

  constructor(){

<strong>    if( this.oidc.initializationError ){
</strong>
<strong>      const { initializationError } = this.oidc;
</strong>
<strong>      // Distinguish a misconfiguration from a temporary auth-server outage.
</strong><strong>      console.log(initializationError.isAuthServerLikelyDown);
</strong>
<strong>      // Developer-only diagnostic with likely cause and fix.
</strong><strong>      // Do not display this to end users.
</strong><strong>      console.log(initializationError.message);
</strong>
<strong>    }
</strong>
  }
}
</code></pre>
{% endtab %}
{% endtabs %}
