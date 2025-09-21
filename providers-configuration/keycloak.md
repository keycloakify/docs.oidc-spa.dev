---
icon: code-simple
---

# Keycloak

{% embed url="https://youtu.be/qJOHjI_QKvk" %}
oidc-spa with Keycloak
{% endembed %}

## Getting the `issuerUri` and `clientId`

`oidc-spa` requires two parameters to connect to your Keycloak instance: `issuerUri` and `clientId`.

```typescript
const { ... } = createOidc({
    issuerUri: "...",
    clientId: "...",
    // ...
});
```

### `issuerUri`

In Keycloak, the OIDC issuer URI follows this format:

**https://**<mark style="color:blue;">**\<KC\_DOMAIN>**</mark><mark style="color:purple;">**\<KC\_RELATIVE\_PATH>**</mark>**/realms/**<mark style="color:green;">**\<REALM\_NAME>**</mark>

* <mark style="color:blue;">**\<KC\_DOMAIN>**</mark>: The domain where your Keycloak server is hosted (e.g., **auth.my-company.com**).
* <mark style="color:purple;">**\<KC\_RELATIVE\_PATH>**</mark>: The subpath under which Keycloak is hosted. In recent versions, this is an empty string (`""`). In older versions, it was `"/auth"`.\
  Check your Keycloak server configuration; this parameter is typically set using an environment variable:\
  Example: `-e KC_HTTP_RELATIVE_PATH=/auth`
* <mark style="color:green;">**\<REALM\_NAME>**</mark>: The name of your realm (e.g., **myrealm**).\
  🔹 **Important:** Always create a dedicated realm for your organization—**never use the master realm**.\
  To create a new realm:
  1. Open **https://**<mark style="color:blue;">**\<KC\_DOMAIN>**</mark><mark style="color:purple;">**\<KC\_RELATIVE\_PATH>**</mark>**/admin/master/console**.
  2. Log in as an administrator.
  3. Click on the realm selector in the top-left corner.
  4. Click **"Create a new Realm"**, give it <mark style="color:green;">a name</mark>, and save.

### `clientId`

The `clientId` is usually something like '<mark style="color:yellow;">myapp</mark>'. Follow these steps to create a suitable client for your SPA:

1. Open **https://**<mark style="color:blue;">**\<KC\_DOMAIN>**</mark><mark style="color:purple;">**\<KC\_RELATIVE\_PATH>**</mark>**/admin/master/console**.
2. Log in as an administrator.
3. Select <mark style="color:green;">your realm</mark> from the top-left dropdown.
4. In the left panel, click **Clients**.
5. Click **Create Client**.
6. Enter a **Client ID**, for example, <mark style="color:yellow;">myapp</mark>, and click **Next**.
7. Ensure **Client Authentication** is **off**, and **Standard Flow** is enabled. Click **Next**.
8. Set two **Valid Redirect URIs,** ensure both URLs end with `/`:
   * **https://**<mark style="color:orange;">**\<APP\_DOMAIN>**</mark><mark style="color:red;">**\<BASE\_URL>**</mark>
   * **http://localhost:\<DEV\_PORT>**<mark style="color:red;">**\<BASE\_URL>**</mark>
   * **Parameters:**
     * <mark style="color:orange;">**\<APP\_DOMAIN>**</mark>: Examples: **https://my-company.com** or **https://app.my-company.com**.\
       🔹 For beter performances ensure <mark style="color:orange;">**\<APP\_DOMAIN>**</mark> and <mark style="color:blue;">**\<KC\_DOMAIN>**</mark> share the same root domain (**my-company.com**). See [end of third party cookies](../resources/end-of-third-party-cookies.md).
     * <mark style="color:red;">**\<BASE\_URL>**</mark>: Examples: **"/"** or **"/dashboard/"**.
     * **\<DEV\_PORT>**: Example: **5173** (default for Vite's dev server, adapt to your setup).
9. Click **Save**, and you're done! 🎉

***

## Session Lifespan Configuration

One important policy to define is how often users need to re-authenticate when visiting your site.

{% hint style="info" %}
This configuration does **not** affect the **access token lifetime** (default: 5 minutes). It controls how long Keycloak keeps **the session active**.
{% endhint %}

### 🔐 Security-Sensitive Apps (Banking, Admin Panels, etc.)

For security-critical apps, users should log in **each visit** and be **logged out** [**after inactivity**](#user-content-fn-1)[^1].

**Why?**\
Users accessing sensitive applications should not remain authenticated indefinitely, especially if they step away from their device. The session idle timeout ensures automatic logout after inactivity.

**Steps to enforce this policy:**

1. **Disable "Remember Me"**:
   * Select <mark style="color:green;">your realm</mark>.
   * Navigate to **Realm Settings** → **Login**.
   * Set **"Remember Me"** to **Off**.
2. **Configure session timeout**:
   * Go to **Realm Settings** → **Sessions**.
   * Set **SSO Session idle**: `5 minutes` (ensures users are logged out after 5 minutes of inactivity).
   * Set **SSO Session max idle**: `14 days` (ensures users who actively use the app don’t get logged out unnecessarily).
3. Optionally, display a logout countdown before automatic logout:

{% content-ref url="../auto-logout.md" %}
[auto-logout.md](../auto-logout.md)
{% endcontent-ref %}

***

### 🛍️ Non-Sensitive Apps (E-commerce, Social Media, etc.)

For apps where users should remain logged in for **weeks or months** (e.g., YouTube-style behavior):

1. **Enable "Remember Me"**:
   * Select <mark style="color:green;">your realm</mark>.
   * Navigate to **Realm Settings** → **Login**.
   * Set **"Remember Me"** to **On**.
2. **Configure session timeout**:
   * Users **without** "Remember Me" will need to log in **every 2 weeks**:
     * Set **Session idle timeout**: `14 days`.
     * Set **Session max idle timeout**: `14 days`.
   * Users **who checked "Remember Me"** should stay logged in for **1 year**:
     * Set **Session idle timeout (Remember Me)**: `365 days`.
     * Set **Session max idle timeout (Remember Me)**: `365 days`.

***

## 🗑️ Allowing Users to Delete Their Own Accounts

By default, Keycloak **does not** allow users to delete their accounts.

If you implement a [delete account button](../user-account-management.md), users will see an **"Action not permitted"** error.

Enabling Account Deletion:

1. Navigate to **Authentication** → **Required Actions**.
2. Enable **"Delete Account"**.
3. Go to **Realm Settings** → **User Registration** → **Default Roles**.
4. Click **Assign Role**, filter by **client**, select **Delete Account**, and assign it.

***

## Testing the Setup

To test your configuration:

```bash
npx degit https://github.com/keycloakify/oidc-spa/examples/tanstack-router-file-based oidc-spa-tanstack-router
cd oidc-spa-tanstack-router
cp .env.local.sample .env.local

# Edit the .env.local file to reflect your configuration

yarn
yarn dev
```

[^1]: The user is considered inactive by oidc-spa when it's not actively moving the mouse, touching the screen or typing on the keyboard in any tab of your app.\
    \
    (More precisely, on any tab of any app that use the same SSO session)
