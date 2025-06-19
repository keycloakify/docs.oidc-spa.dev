---
icon: traffic-light
---

# Why oidcEarlyInit

The standard OIDC flow with PKCE is sometimes perceived as _less secure_ than the traditional "backend for frontend" (BFF) flow, where token exchanges happen on the backend and frontend code never has direct access to tokens.

This perception stems mainly from the nature of JavaScript projects, especially SPAs, which often bundle tens of thousands of npm packages from many different authors.\
If _even one_ of those libraries is compromised, it could potentially steal user tokens unless specific countermeasures are in place.\
There's also the risk of cross-site scripting (XSS) attacks which, if successful, could lead to the same outcome.

The **early init setup** recommended in `oidc-spa` is designed to mitigate these risks.\
By letting a minimal piece of `oidc-spa` run _before_ any other JavaScript code, the library ensures that:

* The response from the authorization server is removed from the URL and stored in memory (scoped variable), so even if malicious code runs later, it can’t access the tokens.
* Critical browser APIs, like `window.fetch`, are frozen to prevent malicious code from intercepting or tampering with authenticated requests.

An added (though minor) benefit of the early init + lazy loading setup is improved performance during silent SSO operations in iframes.\
In the iframe, only a minimal part of the app is evaluated—just the part that needs to forward the auth response to the parent window.\
Since JS files in SPAs are typically hashed and cached, the benefit isn't about download time but about reducing JavaScript _evaluation_ time.\
That said, the gain is usually marginal—just a few extra milliseconds in most cases.

### Is early init mandatory?

No. If not explicitly called, it will be invoked automatically as soon as `oidc-spa` is evaluated, which, in most setups, happens early enough.

### Can it be skipped?

Yes, if:

* The application is free of XSS vulnerabilities.
* The dependencies can be fully trusted.
* Performance at initialization time is not critical.

In such cases, skipping the setup is reasonable.

### Important caveat for React Router (framework mode)

If using **React Router in framework mode**, then the early init setup **must be used**. React Router performs early redirects that can interfere with `oidc-spa`'s initialization under certain circumstances.\
To avoid issues, `oidc-spa` must run first.

If the project is using Vite or Create React App without a higher-level framework, this caveat does not apply.
