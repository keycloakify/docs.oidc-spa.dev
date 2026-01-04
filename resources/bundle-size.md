---
description: Understanding oidc-spa’s impact on your bundle size
icon: scale-unbalanced-flip
---

# Bundle Size

`oidc-spa` ships as a single package.

It includes browser code, server helpers, and multiple adapters. That can make “bundle size” reports look confusing at first.

This page breaks down:

* what ends up in your **initial download**
* why some tools report a much larger “import cost”

### What your app typically downloads

In the common “happy path” (modern browser, secure context), the initial cost is roughly:

* `oidc-spa/entrypoint`: **≈5.2 KB min+gzip**. Runs early to harden the runtime environment.
* `oidc-spa/core`: **≈27.9 KB min+gzip**. The main OIDC implementation.

Total: **≈33 KB min+gzip**.

Add **≈4 KB** if you use higher-level React / Angular adapters.

{% hint style="info" %}
These numbers are “what the browser downloads”, not “what npm installs”.
{% endhint %}

### Why tools sometimes report ≈151 KB

<figure><img src="../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

Tools like “Import Cost” tend to:

* sum **all potentially reachable code**, even if it is split into separate chunks
* ignore whether a chunk is only loaded as a **runtime fallback**

`oidc-spa` generates optional chunks. The biggest ones are usually related to the `crypto.subtle` fallback.

Those chunks are only downloaded for apps that are deployed over `http://` (where `window.isSecureContext === false`).

### Example bundle visualization

<div data-full-width="true"><figure><img src="../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure></div>

This example shows a vanilla Vite app with only `oidc-spa` installed. Notice how the optional polyfills are in separate chunks.
