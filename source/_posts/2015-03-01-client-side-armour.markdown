---
layout: post
title: "Client side armour"
date: 2015-03-01 16:24
comments: true
categories: ["Security", "HTML", "JavaScript", "development"]
---

![Helmet armour](/images/clientsecurity/helmet.jpg)

As web attacks become far more common place, specifications are quickly getting ratified to stop the common attack vectors from being used. W3C has set up a [Security Working Group](http://www.w3.org/Security/) just for this purpose.

The technologies mentioned here are all *belt and braces* security measures also known as **defence in depth** they therefore should not be considered the only defence against attacks.

However for browsers that do support the standards gain from the increased security but browsers that don't are still vulnerable to the attacks.
Even when all browsers do support the technology it is probably always safe to assume that they might not work.


## Pinning behaviour to the browser

The web itself was designed to be stateless and so since lots of statefull tech has been added on top, the new forms of security are in this realm and also called **TOFU security** or trust on first use; obviously all modern tech needs a hipster name.
**TOFU** is actually an old concept that is familiar with SSH, by acknowledging you know a site, you are entrusting it on your first use. Despite it's usage in browsers not as secure, it is another way to prevent a server from being stolen by an attacker after it was first used.

![Tofu](/images/clientsecurity/tofu.jpg)

SSH requests the user to approve unknown servers credentials, however this isn't the case for web mostly I suspect because of the level impact to user experience - it is far from perfect but it does help prevent some attacks of trust.

[DNS poisoning](http://en.wikipedia.org/wiki/DNS_spoofing) and [DNS hijacking](http://en.wikipedia.org/wiki/DNS_hijacking) are two ways in which an attacker can abuse the users trust in the server.

The following three technologies help solve these issues allowing a site to pin behaviour to the browser for future requests:

- [(HKPK) Public key pinning](https://tools.ietf.org/html/draft-ietf-websec-key-pinning-21) - giving browsers the ability to pin what a sites certificates will be should make it much harder for an attacker to game the trust in a servers identity. Preventing man in the middle attacks will become much simpler with this as a client will not trust a response with a different certificate.

- [CSP pinning](http://www.w3.org/TR/2015/WD-csp-pinning-20150226/) - this has just been released as a working draft by the W3C and so there is no intent even for the browsers to implement yet to my knowledge.
However it is exciting that, a browser would be able to know that it can immediately block all requests to certain resources from the site for a certain time window.
This adds further hardening to [CSP2](http://www.w3.org/TR/CSP/) which gives the strongest security for resources loading within a page and also became a recommendation last month.
Perhaps this directive might also lead to CSP directives being able to be loaded externally via some form of manifest file, the header could then just contain the integrity of the file, that way as directives become more complex the overhead doesn't increase exponentially.

- [(HSTS) HTTP strict transport security](https://tools.ietf.org/html/rfc6797) - This isn't new however fits into the same category as the two above. This allows sites to always enforce that they are in HTTPS mode only. Subsequent requests to the site will load HTTPS.

## Preventing clickjacking

[UI security](http://www.w3.org/TR/UISecurity/) is worth mentioning as a new set of directives for securing the interface of an application from malicious behaviour.
These directives are additions to the CSP policy which add the following protections:
- Obstructing page elements and triggering the user to click on items that were not expected.
- Allowing certain elements or behaviours through.
- Preventing time based attacks.
- Preventing elements from being placed near certain elements.
- Adding a DOM interface to expose suspected attacks to JavaScript allowing the application to alert the user.
- Further extending violation reports provided by CSP.

## Fixing insecure resources

[Upgrade Insecure Requests](http://www.w3.org/TR/2015/WD-upgrade-insecure-requests-20150226/) follows on nicely from HSTS, which tells the browser to force all insecure content to be HTTPS. For example the author may not be in control of all the site content and can essentially rewrite the code to have 'https' in URLs that are 'http'.

## Preventing information leaking
[Referrer policy](http://www.w3.org/TR/referrer-policy/)
Provides a way to prevent browsers from supplying a *referer* header to other websites.
This prevents any information leaks of URLs via the header, this reduces the risk of users exposing information from where they came from. It also adds further protection against the risk of unique tokens being exposed; despite not ideal to be in the URL in the first place certain interfaces require it.

## Ensuring our content is correct
[(SRI) Sub Resource Integrity](http://www.w3.org/TR/SRI/) Is a way to ensure that content served to the browser is of a known content shape.
Sites serving integrity attributes prevent browsers from executing the sub resources unless they match the identity specified in the HTML.
This prevents scripts from being overridden with malicious content, it's primary application is content served over a CDN however it doesn't harm making attackers lives harder.

Another use case is restricting the content served from third parties, most sites often load many widgets and scripts from other sources - All of which may not remain secure over time.
The developers of the application may not have a choice over if the resource is loaded on the site, for example it could be a primary revenue stream for the site.
However the developer would be able to run the current code served from the third party and serve the integrity parameter to the script.
If the resource then gets compromised the sites supporting users wouldn't be at risk of the attack.

## Headers still worth using
- [X-XSS-Protection](https://msdn.microsoft.com/en-us/library/dd565647%28v=vs.85%29.aspx) - XSS filtering for common techniques used for injecting XSS. Never standardised however it is acknowledged by both Chrome and Internet Explorer.

- [X-Content-Type-Options](https://msdn.microsoft.com/en-us/library/ie/gg622941%28v=vs.85%29.aspx) - Used to stop browsers from sniffing the content of an asset and running it unless it matches the content type sent from the server. So for example loading `text/plain` content will never be sniffed and loaded as JavaScript.

- [X-Frame-Options](https://tools.ietf.org/html/rfc7034) Is a way of preventing clickjacking, by informing browsers when the site is allowed to be loaded through framing.  
  - This has been integrated within [CSP2 frame ancestors](https://w3c.github.io/webappsec/specs/content-security-policy/#directive-frame-ancestors) however it is still worth providing this until full support is available for **CSP2**.

- [Access-Control-Allow-Origin](http://www.w3.org/TR/cors/#access-control-allow-origin-response-header) - A way of opening up a site to allow cross origin requests, it is worth noting this here as it impacts security. Unless you need CORS functionality then don't return this header.
