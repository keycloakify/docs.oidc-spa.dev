---
description: How oidc-spa mitigates the risks of token exposure
icon: shield-check
---

# Token Exfiltration Defence

oidc-spa implements a comprehensive, defense-in-depth strategy to protect against token exfiltration in case of successfull XSS or supply chain attack.

The goal of these defence is to acheive the same level of security guarenty with client side auth than what you get when implementing auth on the bakend, the classic session based cookie model.  \
\
The concerns addressed by this defences are explained in this talk. None still stands with the exfiltration defence enabled.

{% embed url="https://youtu.be/MpPd0WnEG5s?si=ZwlZujfmYboSMlE-&t=779" %}

When oidc-spa is running with exfiltration defence enabled an attacker cannot request or access tokens. Just like in cookie based auth model.

## Enabling the exfiltration Defence

{% hint style="warning" %}
It's possible that your app won't start once you've enabled the token exfiltration defense.

If it's the case this means that one of the dependency you are using is trying to monkey patch the builtins.

oidc-spa can't allow that to happen while protecting your token from exfiltration.

Example of libraries that are incompatible with oidc-spa defence:

* @microsoft/applicationinsights: Monkey patches fetch
* Zone.js: Monkey patches Promise and XHR

If you are in that situation your two only option are:

* Part way with those libraries.&#x20;
* Run oidc-spa with exfiltration defence disabled.

Note howerver than even with exfiltration defences disabled, oidc-spa still implement all the current best practicies for client side auth. Including zero token persistance.

You app will still pass any security audit you'll throw at it.
{% endhint %}

Enabling the security defences of oidc-spa is just a matter of flipping a switch.

{% tabs %}
{% tab title="Vite Plugin" %}
<pre class="language-typescript" data-title="vite.config.ts"><code class="lang-typescript">import { oidcSpa } from "oidc-spa/vite-plugin";

export default {
  plugins: [
    // ...
    oidcSpa({
<strong>      enableTokenExfiltrationDefense: true,
</strong><strong>      // If you access external resource servers, (other than you own server APIs)
</strong><strong>      // you must declare them.
</strong><strong>      //resourceServersAllowedHostnames: ["vault.my-company.com", "s3.my-company.com"]
</strong>    })
  ]
};
</code></pre>
{% endtab %}

{% tab title="Manual Setup" %}
<pre class="language-typescript" data-title="src/main.ts"><code class="lang-typescript">import { oidcSpaEarlyInit } from "oidc-spa/earlyInit";

oidcSpaEarlyInit({
<strong>    enableTokenExfiltrationDefense: true,
</strong><strong>    // If you access external resource servers, (other than you own server APIs)
</strong><strong>    // you must declare them.
</strong><strong>    //resourceServersAllowedHostnames: ["vault.my-company.com", "s3.my-company.com"],
</strong>});
</code></pre>
{% endtab %}
{% endtabs %}

## Understanding the security guarantee and it's limits

### Supply chain attacks

If you use a compromised version of an NPM dependency, the potential damage are very limitted. The attacker can't exfiltrates token and this is by far the more important security garentee, most NPM supply chain attack are oportunistic, they will try to exfiltrate tokens of as many app as they can and see what they can do with it. You're protected against this.

What's theorially possible is that the attacker manage to perform acction on behafe of the user while the attack is going on. &#x20;

But with oidc-spa, the chances of that happening are vanishingly small. First because this implies that the compromission would be specifically targetting your app, an unless you are building a massively used open source system like Keycloak itself this is just not realistic. &#x20;

But even if the attack would target specifically your system, unlike with traditional session cookie based auth where the attacker can just make any fetch call to the API and have the credential automatically attached to the request, here they would first need to get a hold on the reference of your fetchWithAuth function or getOidc util. Unless you build your app as a single chunk those are usually exposed somewhere but those static assets are hashed (example: assets/KcAdminUi-BV3D797K.js). The hashes are likely to have changed between the time the attacker craft the attack and the time the compromised dependency make it in your bundle. Plus oidc-spa [prevent the discovery of the module graph](#user-content-fn-1)[^1].  \
\
Bottom line: When it comes to supply chain attack you're very well protected, even better protected than with session cookie auth.

### XSS Attacks

XSS attacks on the other hand can still be very damaging. Because an XSS is never oportunistic and always targetting specifically your app. In this threat model you can assume that the attacker knows everything about the module graph and will be able to import your fetchWithAuth function or getOidc util.

They will be able to perform any action the currently logged in user can perform.

Does this means that oidc-spa is less secure than session cookie auth to that regard? No. It's equally unsecure. With session cookie make it even easier because the attacker don't even need to find your fetchWithAuth function in the module graph to start making authed request. But this is irrelevent since an AI assisted, competent attacker will be able to do that. &#x20;

The good news however is that, no solution other than autditing one by one each of your dependency is the only way to prevent supply chain attack, which is not realistic, XSS attack on the other hand can be very effictively blocked by enabiling strict CSP rules. Which you should absolutly do.

Bottom line: XSS attacks are still dangerous, oidc-spa protect agains token exfiltration but not against the attacker acting on behafe of the user. You should still enabled CSP. &#x20;

### Compromised browser extention

This is where cookie based auth still has an edge over oidc-spa in terms of token protection.  \
If one of your user has a compromised browser dependency installed, the extention can monitor network traffic, it will be able to see the tokens going out.  \
But this wouldn't affect every of your user, only the one with the compromised extention. And that wouldn't be your app being compromised for that user but all the SPAs implementing client side auth.

## How odic-spa acheive this

The whole security strategy is build on the fact that, thanks to the Vite plugin or via the oidcEarly init that you'll have setup. oidc-spa has a window of execution where it's garenteed that no other javascript has eveluated yet. &#x20;

During that window it can:

* Harden the environement: Prevent fetch, XHR, Websocket, String, Promise and other critical language builting from being monkey patchet at runtime.&#x20;
* Clear the code response from the auth server present in the url if any and move it safely in memory where it can't be accessed.&#x20;
* Register a message listener that can't be unregistred and that stop the propagation of the auth server response during silent sign in.
* Even if you don't have CSP enabled, only service workers from your origin (or from an accept list) can be loaded.
* And the more important defense of all: Guarentee that the tokens are never actually exposed to the application layer. Tokens exposed to the application layer are unusable for resource server calls.\
  They remain structurally valid JWTs, but their signature segment is replaced.\
  Before any request leaves the app (fetch, XHR, WebSocket, beacon),\
  the real tokens are restored inside a fully hardened, sandboxed interceptor that is implemented in the early init phase.\
  This means that the only way for you to even see the real tokens is to look at the network trafic.

All those mesure have zero impact on DX or performance. They only require that you don't use any library that implement monkey patching of critical language APIs.



[^1]: 
