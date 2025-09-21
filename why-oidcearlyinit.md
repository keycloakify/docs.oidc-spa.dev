---
icon: traffic-light
---

# Why oidcEarlyInit

The standard OIDC flow with PKCE is sometimes perceived as _less secure_ than the traditional "backend for frontend" (BFF) flow, where token exchanges happen on the backend and frontend code never has direct access to tokens.

This perception stems mainly from the nature of JavaScript projects, especially SPAs, which often bundle thousands of npm packages from many different authors.\
If _even one_ of those libraries is compromised, it could potentially steal user tokens unless specific countermeasures are in place.\
There's also the risk of cross-site scripting (XSS) attacks which, if successful, could lead to the same outcome.

The **early init setup** recommended in `oidc-spa` is designed to mitigate these risks.\
By letting a minimal piece of `oidc-spa` run _before_ any other JavaScript code, the library ensures that:

* The response from the authorization server is removed from the URL and stored in memory (scoped variable), so even if malicious code runs later, it can’t access the tokens.
* Critical browser APIs, like `window.fetch`, are frozen to prevent malicious code from intercepting or tampering with authenticated requests.

An added benefit of the early init + lazy loading setup is improved performance and predictability.\
We ensure no collision or race condition between oidc-spa and your client side routing library (react-router, tanstack...). We also can cancel the loading of the app early in iFrame response. &#x20;

### Is early init mandatory?

Yes. In prior version of oidc-spa it was opt-in, but the benefit in terms of perofrmance and security are too important to be ignored anymore. &#x20;
