---
hidden: true
icon: octagon-exclamation
---

# Blocked Monkey Patching

If you are seeing this page, it means that you have enabled [oidc-spa’s custom exfiltration defense mechanisms](../features/token-exfiltration-defence.md), but they conflict with one or more libraries used in your application.

For these defenses to be effective, oidc-spa must ensure that certain sensitive browser APIs (such as `window.fetch`) have not been modified. Unfortunately, there is no reliable way to distinguish between APIs that were monkey-patched by a legitimate dependency and those altered as part of an NPM supply-chain attack. Allowing exceptions would weaken the guarantees provided by the defense, so oidc-spa does not support whitelisting or bypassing these checks.

As a result, there is currently no workaround. If you cannot remove or replace the libraries that interfere with these checks, the only option is to disable the custom exfiltration defenses.

That said, we are actively working on transparent support for DPoP ([Demonstration of Proof of Possession](https://auth0.com/docs/secure/sender-constraining/demonstrating-proof-of-possession-dpop)). DPoP is a protocol-level standard that provides security properties comparable to those offered by oidc-spa’s custom defenses, without relying on runtime integrity checks of browser APIs.

DPoP support is not available yet, but it should be ready within the next few weeks. Once available, you will be able to enable it without making any changes to your codebase. It will be fully transparent. It will not be enabled by default, only because not all resource servers currently support DPoP. However, if you are using oidc-spa server-side validation or a modern, standards-compliant resource server, this should not be an issue.

You can follow the progress of the DPoP implementation here:

{% embed url="https://github.com/keycloakify/oidc-spa/pull/127" %}
