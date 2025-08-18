---
icon: circle
---

# Clerk

{% hint style="warning" %}
Technically, it works now, but there are still a few ways the "OAuth" feature in Clerk needs to be improved to fully comply with the standard so that generic clients work seamlessly.\
The team has been very helpful so far and already fixed the most critical issues.\
Once the remaining problems are addressed, I’ll update this page.

If you want to use it today, here are the required workarounds:

* Set [`noIframe: true`](../resources/iframe-related-issues.md)
* Set [`__unsafe_useIdTokenAsAccessToken: true`](google-oauth.md) if you need the access token to be a JWT (by default, the issued access token is opaque)
* In the Clerk admin, make sure the consent pages are **not** enabled
{% endhint %}

