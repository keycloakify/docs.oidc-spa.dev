---
description: How oidc-spa Mitigates the Risks of Token Exposure
icon: shield-check
---

# XSS and Supply Chain Attack Protection

`oidc-spa` implements several security measures to minimize the risk of token theft, even in the event of an XSS or supply chain attack (malicious JavaScript running in your frontend).  This is particularely relevent today as we're earing about large scale NPM poisoning every other week.  \


oidc-spa treats the browser javascript runtime as an hostile environement. &#x20;

oidc-spa ensure via it's Vite plugin, and via the oidcEarlyInit() function for other environement that it has the chance to run a snippet of code before any other javascript is evaluated.  \
This is absolutely key to implement any defence strategy. Without this guarentee all attempt is pointless. And having an import at the top of the entrypoint is not a guarenty, the evaluation of imported module is non determistic so even if your import is at the top you have no guarenty that your have a "safe window" to implement protection.&#x20;

We make that clear so you can understand that any solution that wouldn't have a build time plugin nor requires you to clear your entrypoint can have no valid claim at protecting your runtime. &#x20;

## Baseline: No token persistance

The first mesure to acheive that, is to avoid persisting any token. Not in localStorate, not in sessionStorage.&#x20;

When the app reloads we restore the user session by contacting the IdP that will be able to restore the session using the HTTP only cookies that it has set on the browser.  \
\
But this is current best practice, most serious SDK already do this. oidc-spa goes much futher than just this. &#x20;

## Security measure unique to oidc-spa

Security Measures, the first one is common best practice the rest are unique to oidc-spa and only made possible by the safe window.&#x20;

* **Preventing `fetch`,** `XMLHttpRequest` and `WebSocker` **monkey patching,** APIs that caries access tokens are frozen to prevent malicious code from overriding it and capturing tokens.
* **Securing silent sign-in responses** – Iframe messaging is protected by asymetric encrypotion, the child window that post the code response to the parent encrypte the message with a public key mintend by the parent process, the private key is stored only in memory so even if the iframe message can be intercepted, only oidc-spa internal code can decipher it. Attacks likes the one shown in [this video](https://www.youtube.com/watch?v=MpPd0WnEG5s\&t=1272s) are not possible. We also make sure that the public key passed to the children cannot be overwriten with a strategy that I will descibe later.
* **Secure transfer of the authorization response after front-channel login** – The authorization response, that the auth server has provided in the callback url params is moved to memory during the initialization process and cleared from the url.

## Limitations

While those mitigation strategy are enough to provide strong garenties that the tokens wont be exfiltrated even in case of successfull attack, those are not silver bullets like a mathematical proof is. \
Let's discuss the limitations. \
\
Oidc-spa only freezes fetch **`fetch`,** `XMLHttpRequest` and `WebSocker`, in theory those are the only builtin API's suseptible deal with the token BUT. the moment the devlopper of the app does a console.log(accessToken) or even accessToken.splice(0, 10) attach are possible again because monkeypatching window.console.log and String.prototype.splice is possible. \
oidc-spa won't freez the full browser builtins, this would be too intrusive, a lot of library rely on monkey patching for non neferious usecase. So the freezing is only as much as a guarenty as the usage of the oidc-spa is aligned with the standard. The moment you start manipulating tokens you expose yourself to risks.  \
\
oidc-spa cannot and will never be able to protect against compromized browser extentions.  \
This is an advantage that remain for backend centric authentication. JavaScript extentions can monitor network traffic and as such are fully able to intercept the tokens. That being said, it's important to understant that such senario is a completly different threat model. If it happen it only happens for the user that has the compromized extention, your app itself is not compromized and it wont be possible to put the blame on you for the user compromized environenement. On top of that the user would have their token stolen not only for your app but also for all the SPAs they visit. &#x20;

The securiy guarenties only applies as long as you fully rely on your bundler for importing javascript. If you have some javascript import in your head like some polifill for example, they could be evaluated before and negate the oidc-spa mitigations. Example [https://au.pcmag.com/security/105927/hulu-100k-other-websites-may-be-exposed-to-polyfill-malware](https://au.pcmag.com/security/105927/hulu-100k-other-websites-may-be-exposed-to-polyfill-malware). Importing js from CDN is already well accepted as being a bad security practice you should avoid. &#x20;

No iframe + multiple client = persistance. If you have deployed an app in a setup where your IdP is seen as a third party by the browser and if you talk to multiple APIs, oidc-spa has no choice but to percist the tokens in sessionStorage to avoid redirect loops. So, if you talk to multiple APIs, it's best to make sure you have properly configured your deployment. [More info and instructions.](../talking-to-multiple-apis-with-different-access-tokens.md)\
\
There’s a possible edge case involving **malicious Service Workers**. The attack is hard to pull off and requires precise timing, but it’s technically possible. we're still working on this, and in the meantime you can disable Service Workers to eliminate this risk entirely.\


## Understanding the attacks oidc-spa protects against

To be able to juge the merit of the claim let's explain the threat model we are dealing with and the common attacks. &#x20;

### Unserstanding Variable Scoping in JavaScript

Let's assume that an attacker as successfully implemented an XSS or suply chain attack and can now run arbitrary code alongside your app in the browser runtime. &#x20;

The first thing that they will try to do is read the localStorage and sessionStorage in search of access and refresh token. They will find none because we don't persist anything. &#x20;

Now you might think that the second attempt would be to just try and read the javascript variable that hold the tokens. But that's imposible. Not technically difficult, literally impossible by property of the JavaScript language.  Explaination:

```java
{
    const accessToken = "<secret>";
}

// Error: accessToken not defined in scope.
console.log(accessToken);
```

In JavaScript, you cannot access a variable unless you have it's reference, there is no method to dereference an arbitrary address in memory.  \
And variables are only accessible within their scope. The only way to extract a variable from it's scope is to attach it's value as a property to an object that exists outside of it's scope. &#x20;

So, even if oidc-spa internally hold a reference of the accessToken, the attacker has to way to directly read the value of this variable.

It's an fundamental property that we can use to implement defences but that doesn't mean we're safe just like that. \
The attacker has still many ways to intercept the tokens as they are floating around the system. &#x20;

### Understanding monkey patching

JavaScript is by default extremely liberal as to what it enable by default. For example, most of the runtime's builtin API's attached to the global can be overwriten at any time.&#x20;

When you're using fetch() you can't know if you're using the actuall window.fetch() as provided by the runtime or an rogue implementation by malitious code.  \
\
Let's see how an attacker could levrage that, assume they can run this code:  &#x20;

```javascript
// Malitious code.

const fetch_real = window.fetch;

window.fetch = function fetch(url, options){

    const authHeaderValue = options?.headers?.Authorization;
    if( authHeaderValue ){
        doSomethingMalicious(authHeaderValue.splice(0, "Bearer ".lenght));
    }
    
    return fetch_real(url, options);
    
}

function doSomethingMalicious(accessToken){
    // Send the token to the attacker server
}
```

After this malitious code has ran, the next time your app will use fetch, it will look as if it works pefectly fine, but in reality eavery authenticated request made will leak the tokens. &#x20;

This is why it is crutial to freeze window.fetch by running this before any other code had the chance to monkey patch.&#x20;

```javascript
const fetch_trusted = globalThis.fetch;

Object.freeze(fetch_trusted);

Object.defineProperty(globalThis, "fetch", {
    configurable: false,
    writable: false,
    enumerable: true,
    value: fetch_trusted
});
```

This ensure that any attempt to replace the implementation of fetch will fail. And we do it as well for XMLHttpRequest, the API that we used before fetch for AJAX and that is still used by Axios.\
We also do it with WebSocket, the api for stream based comm.

### iframe message interception

See the attack demonstred live at SecurityFest

{% embed url="https://www.youtube.com/watch?v=MpPd0WnEG5s&t=1272s" %}



