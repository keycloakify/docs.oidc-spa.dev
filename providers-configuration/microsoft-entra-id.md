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
6. Set **Supported Account Type** to **Accounts in this organization**.
7. In the left menu, go to **"Manage > Expose API"**.
8. Click **"Add a scope"**.
9. Configure as follows, then click **"Add Scope"**:
   * **Application ID URI**: `api://my-app-api` (then save and continue)
   * **Scope name**: `access_as_user`
   * **Who can consent**: Admins and Users
   * **Admin Consent Display Name**: "JWT Access Token"
   * **Admin Consent Description**: "Read permission on the basic user profile"
   * **User Consent Display Name**: "View your basic profile"
   * **User Consent Description**: "Allows the app to see your basic profile (e.g., name, picture, user name, email address)"
   * **State**: Enabled

***

## Registering Your Application

1. Go to [Microsoft Azure Portal](https://portal.azure.com/).
2. In the left panel, select **"Microsoft Entra ID"**.
3. Navigate to **"Manage > App Registrations"**.
4. Click **"New Registration"**.
5. Enter **"My App"** as the display name (replace with your actual app name).
6. Set **Supported Account Type** to [**Accounts in this organization**](#user-content-fn-1)[^1].
7. Click **Register**.
8. Click **"Add a Redirect URI"**.
9. Click **"Add Platform"** > **"Single-Page Application"**.
10. Set **Redirect URIs**:
    * **Production**: `https://my-app.com/` (include trailing slash; adjust if hosted under a subpath, e.g., `https://my-app.com/dashboard/`)
    * **Local Development**: `http://localhost:5173/` (include trailing slash; adjust based on your dev server)
11. Ensure **"Access Token"** and **"ID Token"** are checked.
12. Click **Save**.
13. In the left panel, go to **"API Permissions"**.
14. Click **"Add a permission"**.
15. Click **"APIs My Organization Uses"**.
16. Select **"My App - API"**.
17. Check **"access\_as\_user"**, then click **"Add permission"**.
18. In the left panel, click **"Overview"** and copy:
    * **Application (client) ID**
    * **Directory (tenant) ID**

These are required to configure `oidc-spa`.

***

## Providing the parameters to oidc-spa

{% include "../.gitbook/includes/entra-id-config.md" %}

[^1]: Only for now. You can change that later if you want to enable pepole to signin with their personal accounts.
