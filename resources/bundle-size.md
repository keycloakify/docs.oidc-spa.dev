---
description: Understanding the impact on oidc-spa on your bundle size
icon: scale-unbalanced-flip
---

# Bundle Size

Because oidc-spa is a single package that bundles adapter and utils both for the client and the backend and for multiple framwork it can be tricky to assess the actuall inpact of oidc-spa on your bundle size.

So let's see in details:

* oidc-spa/entrypoint - 5.2KB min+gzip. The crutial code required to harden the environement early.
* oidc-spa/core - 27.9KB - The actuall implementation of oidc-spa

In total it's about \~ 33kb that is downloaded in normal circumstances. Add \~4kb if you use higher level React / Angular adapters. &#x20;

Why is it then that import cost is reporting oidc-spa to wheigh 151KB ?

<figure><img src="../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

Because oidc-spa generate optional chunks that will be downloaded as fallbacks, the bigger ones are related to the crypto.subtle polyfill. But those polifill won't be actually downloaded unless your app is deployed without SSL (which can be the case in some intranet environement).

Here is a visualization with bundle-analizer of a vanilla vite app with only oidc-spa installed:

<div data-full-width="true"><figure><img src="../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure></div>
