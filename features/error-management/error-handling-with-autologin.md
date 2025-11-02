# Error Handling - With AutoLogin

In Auto Login mode, error handling works a little differently since we have no mode to fallback to in case oidc-spa failed to initialize.&#x20;

We expose sprate hooks to let you gracefully handle error separatly:

{% tabs %}
{% tab title="Framework Agnostic" %}
```typescript
import { createOidc, type OidcInitializationError } from "oidc-spa/core";

const oidc = await createOidc({
    // ...
    autoLogin: true
})
// NOTE: in autoLogin: false mode, createOidc can never throw (or it's on us)
// In autoLogin: true it can throw but only OidcInitializationError so you can
// safely cast here.
.catch(error => error as OidcInitializationError);

if( oidc instanceof Error ){

    const oidcInitializationError = oidc;
    
    // This help you discriminate configuration errors
    // and error due to the server being temporarely down.
    console.log(initializationError.isAuthServerLikelyDown);
    
    // This is a debug message that tells you what's wrong
    // with your configuration and how to fix it.
    // (this is not something you want to display to the user)
    console.log(initializationError.message);
    
    alert("Our auth is down, sorry :(");
    
    await Promise<never>(()=>{});
}
```
{% endtab %}

{% tab title="TanStack Start" %}
<pre class="language-tsx" data-title="src/routes/__root.tsx"><code class="lang-tsx">import { HeadContent, Scripts, createRootRoute } from "@tanstack/react-router";

import Header from "@/components/Header";
import { AutoLogoutWarningOverlay } from "@/components/AutoLogoutWarningOverlay";
import { OidcInitializationGate } from "@/oidc";
<strong>import type { OidcInitializationError } from "oidc-spa/core";
</strong>
export const Route = createRootRoute({
    // ...
    shellComponent: RootDocument
});

function RootDocument({ children }: { children: React.ReactNode }) {
    return (
        &#x3C;html lang="en">
            &#x3C;head>
                &#x3C;HeadContent />
            &#x3C;/head>
            &#x3C;body>
                &#x3C;div className="min-h-screen flex flex-col">
                    &#x3C;Header />
                    &#x3C;main className="flex flex-1 flex-col">
                        &#x3C;OidcInitializationGate 
                            pendingComponent={()=> &#x3C;Spinner />}
<strong>                            errorComponent={OidcErrorComponent}
</strong>                        >
                            {children}
                        &#x3C;/OidcInitializationGate>
                    &#x3C;/main>
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
</strong><strong>    // This help you discriminate configuration errors
</strong><strong>    // and error due to the server being temporarely down.
</strong><strong>    console.log(oidcInitializationError.isAuthServerLikelyDown);
</strong><strong>
</strong><strong>    // This is a debug message that tells you what's wrong
</strong><strong>    // with your configuration and how to fix it.
</strong><strong>    // (this is not something you want to display to the user)
</strong><strong>    console.log(oidcInitializationError.message);
</strong><strong>
</strong><strong>    return &#x3C;h1>Our auth is down, sorry&#x3C;/h1>;
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
</strong><strong>    // This help you discriminate configuration errors
</strong><strong>    // and error due to the server being temporarely down.
</strong><strong>    console.log(oidcInitializationError.isAuthServerLikelyDown);
</strong><strong>
</strong><strong>    // This is a debug message that tells you what's wrong
</strong><strong>    // with your configuration and how to fix it.
</strong><strong>    // (this is not something you want to display to the user)
</strong><strong>    console.log(oidcInitializationError.message);
</strong><strong>
</strong><strong>    return &#x3C;h1>Our auth is down, sorry&#x3C;/h1>;
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
</strong><strong>
</strong><strong>      const { initializationError } = this.oidc;
</strong><strong>
</strong><strong>      // This help you discriminate configuration errors
</strong><strong>      // and error due to the server being temporarely down.
</strong><strong>      console.log(initializationError.isAuthServerLikelyDown);
</strong><strong>
</strong><strong>      // This is a debug message that tells you what's wrong
</strong><strong>      // with your configuration and how to fix it.
</strong><strong>      // (this is not something you want to display to the user)
</strong><strong>      console.log(initializationError.message);
</strong><strong>
</strong><strong>    }
</strong>
  }
}
</code></pre>
{% endtab %}
{% endtabs %}
