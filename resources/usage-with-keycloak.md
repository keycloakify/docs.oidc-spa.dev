---
description: Let's spin up a Keycloak server and configure it for your webapp!
---

# 🔑 Usage with Keycloak

### Provisioning a Keycloak server

If you already have access to a Keycloak server you can skip this section. &#x20;

{% tabs %}
{% tab title="I deploy Keycloak myself" %}
Follow one of the following guides: &#x20;

{% embed url="https://www.keycloak.org/getting-started/getting-started-zip" %}
Get started with Keycloak on bare metal
{% endembed %}

{% embed url="https://www.keycloak.org/getting-started/getting-started-docker" %}
Get started with Keycloak on Docker
{% endembed %}

{% embed url="https://www.keycloak.org/getting-started/getting-started-kube" %}
Get started with Keycloak on Kubernetes
{% endembed %}

{% embed url="https://www.keycloak.org/getting-started/getting-started-openshift" %}
Get started with Keycloak on OpenShift
{% endembed %}

{% embed url="https://www.keycloak.org/getting-started/getting-started-podman" %}
Get started with Keycloak on Podman
{% endembed %}
{% endtab %}

{% tab title="Managed Keycloak instance" %}
Don't want to deploy and maintain a own Keycloak server yourself?&#x20;

Choosing Keycloak as a Service through a cloud IAM provider can offload the complexities of management and maintenance. It ensures that your system is always up-to-date with the latest security patches and features without the direct overhead of server upkeep. This is especially beneficial for teams prioritizing development and innovation over infrastructure management, offering robust support and service level agreements to guarantee smooth operation. &#x20;

{% embed url="https://www.cloud-iam.com/" %}
{% endtab %}
{% endtabs %}

### Configuring your Keycloak server

Let's configure your Keycloak server with good default for an SPA.&#x20;

Connect to the admin panel of your Keycloak server (we assumes it's https://auth.my-domain.net/auth)

* Create a realm called "myrealm" (or something else), go to **Realm settings**
  1. On the tab General
     1. _User Profile Enabled_: **On**
  2. On the tab **login**
     1. _User registration_: **On**
     2. _Forgot password_: **On**
     3. _Remember me_: **On**
  3. On the tab **email,** we give an example with [AWS SES](https://aws.amazon.com/ses/), if you don't have a SMTP server at hand you can skip this by going to **Authentication** (on the left panel) -> Tab **Required Actions** -> Uncheck "set as default action" **Verify Email**. Be aware that with email verification disable, anyone will be able to sign up to your service.
     1. _From_: **noreply@my-domain.net**
     2. _Host_: **email-smtp.us-east-2.amazonaws.com**
     3. _Port_: **465**
     4. _Authentication_: **enabled**
     5. _Username_: **\*\*\*\*\*\*\*\*\*\*\*\*\*\***
     6. _Password_: **\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\***
     7. When clicking "save" you'll be asked for a test email, you have to provide one that correspond **to a pre-existing user** or you will get a silent error and the credentials won't be saved.
  4. On the tab Themes
     1. _Login theme_: keycloak (see next section to see how to create a theme for your app)
     2. _Email theme_: keycloak
  5. On the tab **Localization**
     1. _Internationalization_: **Enabled**
     2. _Supported locales_: \<Select the languages you wish to support>
  6. On the tab **Sessions**
     1. SSO Session Idle: 14 days - This setting and the following two are so that when the user click "remember me" when he logs in, he doesn't have to login again for the next two weeks.
     2. SSO Session Idle Remember Me: 14 days
     3. SSO Session Max Remember Me: 14 days
*   Create a new OpenID Connect client called "myclient" (or something else) by accessing Clients -> Create Client

    1. _Root URL_: **https://your-domain.net** (or something else, your app does not need to be on the&#x20;

    the same domain as your Keycloak).

    1. _Valid redirect URIs_: **https://onyxia.my-domain.net/\*, http://localhost\* (for testing in local)**
    2. _Web origins_: **\***
    3. Login theme: keycloak (or your theme if you have one)
* (OPTIONAL) In **Authentication** (on the left panel) -> Tab **Required Actions** enable and set as default action **Therms and Conditions.** (You can use Keycloakify to specify your therme and condition, see next section)
* (OPTIONAL) On the left pannel you can go to identity provider to enable login via Google, GitHub, Instagram, ect...&#x20;



{% hint style="success" %}
Now the parameter that you will have to provide to oidc-spa are:&#x20;

```
    issuerUri: "https://auth.your-domain.net/realms/myrealm",
    clientId: "myclient"
```

Replace `your-domain.net`, `myrealm` and `myclient` by what you actually used in the configuration process.
{% endhint %}

### Customizing the login and register page&#x20;

By default, Keycloak comes with very generic login and register pages, you might want to customize them to match the design system of your App.  \
We've createad a lib that let you create Keycloak theme using React!   &#x20;

{% embed url="https://www.keycloakify.dev/" %}
