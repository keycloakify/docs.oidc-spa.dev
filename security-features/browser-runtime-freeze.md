---
description: Ensuring the integrity of the Browser Runtime Environement.
icon: igloo
---

# Browser Runtime Freeze

{% hint style="danger" %}
Still under construction
{% endhint %}

This is maybe the most important security defence as it's a prerequisite to ensure the other measures are actually effective.

It consist in hardening the Browser environement before any code has been evaluated to make sure that an attaker cannot alterate the behavior of the core javascript language feature to extract token/mint new ones.  <br>

## Understanding the defence

In javascript runtime, by default prety much any global APIs can be alterated at runtime. &#x20;









It’s possible that your app will refuse to start after enabling the defence.

If this happens, a dependency in your app is attempting to monkey-patch critical built-ins.\
oidc-spa cannot allow this while guaranteeing token protection.

Examples of incompatible libraries:

* `@microsoft/applicationinsights`, monkey-patches `fetch`
* `Zone.js`, monkey-patches `Promise` and `XMLHttpRequest`

If you encounter this situation, your only options are:

* Remove or replace the incompatible libraries, **or**
* Disable the oidc-spa exfiltration defence

Even with this defence disabled, oidc-spa still implements all current best practices for secure client-side auth (including **zero token persistence**).\
Your app will still pass a security audit.
