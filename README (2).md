---
icon: up
---

# v7 -> v8

No breaking changes except that the [early init (oidc-spa/entrypoint) is no longer optional](https://app.gitbook.com/u/KU4TxrubpKQ9dtKhYh0SCgWYAZH3).



Beyond that, this release get rid of an unessesary redirect when logging in, this yield a faster login experience. Previously you would have a couple of grey fram after the auth server redirected your users to your app. It's no longer the case.&#x20;
