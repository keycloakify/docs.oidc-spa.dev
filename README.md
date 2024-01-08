# Overview

An OIDC client tailored for Single Page Applications, particularly suitable for [Vite](https://vitejs.dev/) projects.\
This library is intended for scenarios such as integrating your application with Keycloak.

* 🎓 Accessible to all skill levels; no need to be an OIDC expert.
* 🛠️ Easy to set up; eliminates the need for creating special /login /logout routes.
* 🎛️ Minimal API surface for ease of use.
* ✨ Robust yet optional React integration.
* 📖 Comprehensive documentation and project examples: End-to-end solutions for authenticating your app.
* 🧠 Best in class type safety: Enhanced API response types based on usage context.

### Comarison with alternatives

#### [oidc-client-ts](https://github.com/authts/oidc-client-ts)

While `oidc-client-ts` serves as a comprehensive toolkit, our library aims to provide a simplified, ready-to-use adapter that will pass any security audit and that will just work out of the box on any browser.\
We utilize `oidc-client-ts` internally but abstract away most of its intricacies.

#### [react-oidc-context](https://github.com/authts/react-oidc-context)

Our library takes a modular approach to OIDC and React, treating them as separate concerns that don't necessarily have to be intertwined.\
At its core, `oidc-spa` is a straightforward adapter that isn't tied to any specific UI framework, making it suitable for projects that enforce a strict separation of concerns between the core logic of the application and the UI.\
Additionally, we provide adapters for React and starter projects for integration with `react-router-dom` or `@tanstack/react-router`.

#### [keycloak-js](https://www.npmjs.com/package/keycloak-js)

Beside the fact that this lib only works with Keycloak [it is also likely to be deprecated](https://www.keycloak.org/2023/03/adapter-deprecation-update).
