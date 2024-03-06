# 🍪 End of third-party cookies

Google is ending third-party cookies for all Chrome users in 2024 and are already disabled by default in Safari. &#x20;

Let's see how it might affect you. &#x20;

First of all, if your identity server and your app shares the same root domain you are not affected. &#x20;

Example, if you are in the case: &#x20;

* Your app is hosted at www.example.com or dashboard.example.com
* Your identity server, for example Keycloak, is hosted at: auth.example.com

You are not affected ✅. Indeed Both www.example.com, dashboard.example.com and auth.example.com shares the same root domain: example.com.\
\
On the other end, if you are in the folowing case:

* You app is hosted at www.examples.com or dashboard.example.com
* Your identity server is hosted at: auth.sowhere-else.com

Let's see how third party cookies phase out will affect you:&#x20;

* You will see a console warning "Third-party cookie will be blocked" in the console in production. &#x20;
* If a user that is authenticated close the tab of your app or close the browser and open your site again a while later. With third party cookies enabled and assuming he's session haven't expired yet he will be automaticall logged in. With third party cookies disabled your website will load in unautenticated mode. If he clicks on the login button this will trigger a full reload and he will be authenticated without having to enter he's credential again.   &#x20;

