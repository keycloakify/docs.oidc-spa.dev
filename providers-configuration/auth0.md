---
icon: shield-quartered
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/oygeayjvIPxroUcp3jt4/providers-configuration/auth0
---

# Auth0

{% hint style="danger" %}
Recent Auth0 platform changes can require consent during silent authentication. This breaks token rotation for affected SPA integrations. We cannot currently recommend Auth0 for SPAs.\
See the [Auth0 community report](https://community.auth0.com/t/silent-authentication-systematically-requiring-consent-in-spa-breaking-token-rotation-for-oidc-spa-library-users/201575).\
\
On top of this DPoP needs to be [explicitely disabled](../security-features/dpop.md#enabling-dpop) because Auth0 claim to support it but does not in a standard way.&#x20;
{% endhint %}

{% embed url="https://www.youtube.com/embed/zPikliLzC84?si=_bIUM5lxNwDIZ3eR" %}

## Configuring a Custom Domain

First step is to configure a custom Domain.

1. Navigate to the [Auth0 Dashboard](https://manage.auth0.com/dashboard).
2. Click **Settings** in the left panel.
3. Open the **Custom Domain** tab.
4. Configure a custom domain (e.g., `auth.my-company.com`). Make sure it's a sub domain of where your app will be deployed.
5. Copy this (`auth.my-company.com`) it is your `issuerUri`.

<figure><img src="../.gitbook/assets/image.png" alt=""><figcaption><p>In this screenshot we used auth0.oidc-spa.dev (instead of auth.my-company.com)</p></figcaption></figure>

## Declaring Your Application

1. Navigate to [Auth0 Dashboard](https://manage.auth0.com/dashboard).
2. In the left panel, go to **Applications → Applications**.
3. Click **Create Application**.
4. Select **Single Page Application** as the application type.
5. Scroll to the Application URIs section. Set two **Allowed Callback URLs**:
   * `https://my-app.my-company.com/` (include trailing slash; adjust if hosted under a subpath, e.g., `https://my-company.com/my-app/`)
   * `http://localhost:5173/` (include trailing slash; adjust based on your dev server)
6. **Allowed Logout URLs**: Copy paste what you put into **Allowed Callback URLs**
7. **Allowed Web Origins** and **Allowed Origins (CORS):** The origins of the Callback URLs
8. Click **Save Changes**
9. Copy the **Client ID**

<figure><img src="../.gitbook/assets/image (16).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (12).png" alt=""><figcaption></figcaption></figure>

## Creating an API

If you need Auth0 to issue a JWT access token for your API, follow these steps:

1. Navigate to [Auth0 Dashboard](https://manage.auth0.com/dashboard).
2. In the left panel, go to **Applications → APIs**.
3. Click **Create API**.
4. Navigate to the Settings tab
5. Fill up **Identifier**: Ideally, use your API's root URL (e.g., `https://myapp.my-company.com/api`). However, this is just an identifier, so any unique string works. Copy it, it is your audience (aud claim in the access token's JWT)
6. Under **Access Token Settings: We want to reduce the lifespan of the access token, the 24 hour default is non acceptable for an SPA usecase.**
   * **Maximum Access Token Lifetime**: `5 minutes` (300 seconds), can be even shorter. It only need to be valid for the duration of transit from the frontend to the backend.
   * **Implicit/Hybrid Flow Access Token Lifetime**: `5 minutes` – required to save settings, even if unused.
7. Under the Application Access tab: Click on the edit button on the line of the Application we've created in the previous step (ex: My App), Under "User Delegated Access", click the "Grant Access" button. &#x20;
8. Click **Save**

<figure><img src="../.gitbook/assets/image (17).png" alt=""><figcaption></figcaption></figure>

## (Optional) Configuring Auto Logout

If you want users to be [**automatically logged out**](../features/auto-logout.md) after a period of inactivity, follow these steps.

### When and Why Enable Auto Logout?

For **security-critical applications** like banking or admin dasboards users should:

* Log in **on every visit**.
* Be **logged out after inactivity**.

This prevents unauthorized access if a user steps away from their device.

For apps like social media or e-comerce shop on the other hand it's best **not** to enable auto logout.

### Configuring Session Expiration in Auth0

1. Navigate to [Auth0 Dashboard](https://manage.auth0.com/dashboard).
2. Click **Settings** in the left panel.
3. Open the **Advanced** tab.
4. Configure **Session Expiration**:
   * **Idle Session Lifetime**: `30 minutes` (1800 seconds) – logs out inactive users. Copy the value in seconds this will be your `idleSessionLifetimeInSeconds`.
   * **Maximum Session Lifetime**: `14 days` (20160 minutes) – ensures active users stay logged in.

Since Auth0 **does not issue refresh tokens** (or issues non-JWT ones), inform `oidc-spa` of your settings:

## Providing the Parameters to oidc-spa

{% tabs %}
{% tab title="Framwork Agnostic" %}
{% code title="src/oidc.ts" %}
```typescript
createOidc({
    issuerUri: "auth.my-company.com",
    clientId: "DzXSmwQS7oSTQGLbafhrPXYLT0mOMyZD"
    extraQueryParams: {
       audience: "https://app.my-company.com/api"
    },
    // Auth0 puts DPoP behind a paywall. Explicitely disable it until you have 
    // enabled it in the Auth0 dashboard.
    disableDPoP: true,
    // (Optional) This must be kept in sync with the Idle Session Lifetime value 
    // configured in the Auth0 dashboard. To ensure correct autoLogout behavior.
    idleSessionLifetimeInSeconds: 1800,
    
    // ...
});
```
{% endcode %}


{% endtab %}

{% tab title="React" %}
<pre class="language-typescript" data-title="src/oidc.ts"><code class="lang-typescript">bootstrapOidc({
    issuerUri: "auth.my-company.com",
    clientId: "DzXSmwQS7oSTQGLbafhrPXYLT0mOMyZD"
    extraQueryParams: {
       audience: "https://app.my-company.com/api"
    },
    // Auth0 puts DPoP behind a paywall. Explicitely disable it until you have 
    // enabled it in the Auth0 dashboard.
    disableDPoP: true,
    // (Optional) This must be kept in sync with the Idle Session Lifetime value 
    // configured in the Auth0 dashboard. To ensure correct autoLogout behavior.
    idleSessionLifetimeInSeconds: 1800,
    // ...
});

// In TanStack Start: 
    .withAccessTokenValidation({
        type: "RFC 9068: JSON Web Token (JWT) Profile for OAuth 2.0 Access Tokens",
<strong>        expectedAudience: () => "https://app.my-company.com/api",
</strong>        // ...
    })
</code></pre>
{% endtab %}

{% tab title="Angular" %}
{% code title="src/app/app.config.ts" %}
```typescript
Oidc.provide({
    issuerUri: "auth.my-company.com",
    clientId: "DzXSmwQS7oSTQGLbafhrPXYLT0mOMyZD"
    extraQueryParams: {
       audience: "https://app.my-company.com/api"
    },
    // Auth0 puts DPoP behind a paywall. Explicitely disable it until you have 
    // enabled it in the Auth0 dashboard.
    disableDPoP: true,
    // (Optional) This must be kept in sync with the Idle Session Lifetime value 
    // configured in the Auth0 dashboard. To ensure correct autoLogout behavior.
    idleSessionLifetimeInSeconds: 1800,
})
```
{% endcode %}
{% endtab %}
{% endtabs %}

## Production vs Development

When developing your app in localhost you may notice that your auth status is lost upon reloading the page and that Auth0 asks you to Accept Consent again and again.  \
It's anoying but it only happens in development, not in production.   \
See this note in Auth0 documentation: [https://auth0.com/docs/get-started/applications/third-party-applications/user-consent-and-third-party-applications?utm\_source=chatgpt.com#skip-consent-for-first-party-applications](https://auth0.com/docs/get-started/applications/third-party-applications/user-consent-and-third-party-applications?utm_source=chatgpt.com#skip-consent-for-first-party-applications)

<figure><img src="../.gitbook/assets/image (21).png" alt="" width="375"><figcaption></figcaption></figure>
