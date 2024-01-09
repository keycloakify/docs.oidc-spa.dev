---
description: A solution to implement user authentication in your webapplication
---

# ⭐ Overview

An OIDC client tailored for Single Page Applications, particularly suitable for [Vite](https://vitejs.dev/) projects.\
This library is intended for scenarios such as integrating your application with [Keycloak](https://www.keycloak.org/). &#x20;

In straightforward terms, this library is ideal for those seeking to enable user login/registration in their web application. When used in conjunction with Keycloak (for example), it enables you to offer a modern and secure authentication experience with minimal coding effort. This includes options for signing in via Google, X, GitHub, or other social media platforms. We provide comprehensive guidance from beginning to end.

* 🎓 Accessible to all skill levels; no need to be an OIDC expert.
* 🛠️ Easy to set up; eliminates the need for creating special `/login` `/logout` routes.
* 🎛️ Minimal API surface for ease of use.
* ✨ Robust yet optional React integration.
* 📖 Comprehensive documentation and project examples: End-to-end solutions for authenticating your app.
* ✅ Best in class type safety: Enhanced API response types based on usage context.

### Compared to alternatives

#### [oidc-client-ts](https://github.com/authts/oidc-client-ts)

While `oidc-client-ts` serves as a comprehensive toolkit, our library aims to provide a simplified, ready-to-use adapter that will pass any security audit and that will just work out of the box on any browser.\
We utilize `oidc-client-ts` internally but abstract away most of its intricacies.

#### [react-oidc-context](https://github.com/authts/react-oidc-context)

Our library takes a modular approach to OIDC and React, treating them as separate concerns that don't necessarily have to be intertwined.\
At its core, `oidc-spa` is a straightforward adapter that isn't tied to any specific UI framework, making it suitable for projects that enforce a strict separation of concerns between the core logic of the application and the UI.\
Additionally, we provide adapters for React and starter projects for integration with `react-router-dom` or `@tanstack/react-router`.

#### [keycloak-js](https://www.npmjs.com/package/keycloak-js)

Beside the fact that this lib only works with Keycloak [it is also likely to be deprecated](https://www.keycloak.org/2023/03/adapter-deprecation-update). &#x20;

### Notable project using the library

Onyxia:&#x20;

* [Source code](https://github.com/InseeFrLab/onyxia)
* [Public instance](https://datalab.sspcloud.fr)

{% embed url="https://www.youtube.com/watch?v=FvpNfVrxBFM" %}

The French Interministerial Base of Free Software:&#x20;

* [Source code](https://github.com/codegouvfr/sill-web/)
* [Deployment of the website](https://sill-preprod.lab.sspcloud.fr/)

{% embed url="https://www.youtube.com/watch?v=AT3CvmY_Y7M" %}
