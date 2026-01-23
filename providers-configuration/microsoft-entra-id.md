---
description: Formerly Azure Active Directory
icon: microsoft
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/oygeayjvIPxroUcp3jt4/providers-configuration/microsoft-entra-id
---

# Microsoft Entra ID

{% embed url="https://youtu.be/upcAmYq4JLY" %}

## Declaring your Backend API

This step is important so that the access token issued by Entra ID are in JWT format and specificially crafter for for your backend API.

1. Go to [Microsoft Azure Portal](https://portal.azure.com/).
2. In the left panel, select **"Microsoft Entra ID"**.
3. Navigate to **"Manage > App Registrations"**.
4. Click **"New Registration"**.
5. Enter **"My App - API"** as the name, then click **Register**.
6. In the left menu, go to **"Manage > Expose API"**.
7. Click **"Add a scope"**.
8. Configure as follows, then click **"Add Scope"**:
   * **Application ID URI**: `api://my-app-api` (then save and continue)
   * **Scope name**: `access_as_user`
   * **Who can consent**: Admins and Users
   * **Admin Consent Display Name**: "View user basic profile"
   * **Admin Consent Description**: "Read permission on the basic user profile"
   * **User Consent Display Name**: "View your basic profile"
   * **User Consent Description**: "Allows the app to see your basic profile (e.g., name, picture, user name, email address)"
   * **State**: Enabled

{% hint style="info" %}
The **Application (client) ID** if this App Registration will be the audience claim (aud) that you will need to provide to your backend token validation API.
{% endhint %}

***

## Registering Your Application

1. Go to [Microsoft Azure Portal](https://portal.azure.com/).
2. In the left panel, select **"Microsoft Entra ID"**.
3. Navigate to **"Manage > App Registrations"**.
4. Click **"New Registration"**.
5. Enter **"My App"** as the display name (replace with your actual app name).
6. Click **Register**.
7. Click **"Add a Redirect URI"**.
8. Click **"Add Platform"** > **"Single-Page Application"**.
9. Set **Redirect URIs**: Add at least two
   * **Production**: `https://my-app.com/` (include trailing slash; adjust if hosted under a subpath, e.g., `https://my-app.com/dashboard/`)
   * **Local Development**: `http://localhost:5173/` (include trailing slash; adjust based on your dev server)
10. Click **Save**.
11. In the left panel, go to **"API Permissions"**.
12. Click **"Add a permission"**.
13. Click **"APIs My Organization Uses"**.
14. Select **"My App - API"**.
15. Check **"access\_as\_user"**, then click **"Add permission"**.
16. In the left panel, click **"Overview"** and write down somewhere:
    * `CLIENT_ID=<Application (client) ID>`&#x20;
    * `DIRECTORY_ID=<Directory (tenant) ID>`

These are required to configure `oidc-spa`.

***

## Providing the parameters to oidc-spa

{% include "../.gitbook/includes/entra-id-config.md" %}
