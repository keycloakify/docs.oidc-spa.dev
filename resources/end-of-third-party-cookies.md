---
icon: cookie
---

# End of third-party cookies

Third-party cookies are now blocked on most browsers, including **Chrome**, **Safari**, and **Firefox**.

> Your OIDC provider is considered a third party relative to your application if they do not share a common parent domain.

## How Does This Affect You?

Even if your OIDC provider is treated as a third party by the browser, **in most cases, this does not impact functionality.**\
`oidc-spa` works seamlessly even if auth cookies are blocked.

However, if your app **is allowed** to set cookies on your OIDC provider’s domain, you may see a **slight performance improvement** during the initial load. This is because we can use silent sign-in via an iframe instead of requiring a full page reload.

The only scenario where blocked cookies significantly degrade the user experience is when **both** of the following conditions are true:

* Your OIDC provider does **not** issue refresh tokens.
* The access token has a short lifespan.

For example, if the access token expires every 20 seconds, your app will be forced to reload every 18 seconds, which is not ideal.

To avoid this issue, ensure that your OIDC provider **shares a common parent domain** with your app so that browsers do not treat it as a third party.

## When Are Cookies Considered Third-Party?

> Third-party cookie restrictions apply when your OIDC provider and your application **do not** share a parent domain.

### Examples:

✅ **No third-party cookie issues** (Same Parent Domain)

* App hosted at `www.my-company.com`, `dashboard.my-company.com`, or `my-company.com/dashboard`
* `issuerUri`: `https://auth.my-company.com/realms/myrealm`
* **Parent domain:** `my-company.com`

❌ **Cookies blocked as third party** (Different Parent Domains)

* App hosted at `my-company.com`
* `issuerUri`:
  * `https://accounts.google.com`
  * `https://xxxx.us.auth0.com`
  * `https://login.microsoftonline.com/xxx/v2.0`
  * `https://hydra.project-name.ory.cloud/`
* **No common parent domain → Third-party cookies will be blocked.**

***

## Google reCAPTCHA

While reCAPTCHA is not directly related to `oidc-spa`, its cookies are set on the **login page**, outside of your app. Since this is a related concern, you can find more details here:

{% embed url="https://docs.keycloakify.dev/faq/google-recaptcha-and-end-of-third-party-cookies" %}
