---
hidden: true
icon: octagon-exclamation
---

# Blocked Monkey Patching

If you are seeing this page, it means that you have enabled [oidc-spa’s custom exfiltration defense mechanisms](../security-features/overview.md), but they conflict with one or more libraries used in your application.

For these defenses to be effective, oidc-spa must ensure that certain sensitive browser APIs (such as `window.fetch`) have not been modified. Unfortunately, there is no reliable way to distinguish between APIs that were monkey-patched by a legitimate dependency and those altered as part of an NPM supply-chain or XSS attack. Allowing exceptions would weaken the guarantees provided by the defense, so oidc-spa does not support whitelisting or bypassing these checks.

As a result, there is currently no workaround. If you cannot remove or replace the libraries that interfere with these checks, the only option is to disable the custom exfiltration defenses.

However: You can probably enable DPoP and get most of the security benfit you get when enabling oidc-spa's custom security defences. &#x20;

{% content-ref url="../security-features/dpop.md" %}
[dpop.md](../security-features/dpop.md)
{% endcontent-ref %}
