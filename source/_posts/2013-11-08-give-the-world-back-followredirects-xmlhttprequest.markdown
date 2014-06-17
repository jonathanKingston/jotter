---
layout: post
title: "Give the world back FollowRedirects XMLHttpRequest"
date: 2013-11-08 18:54
comments: true
categories: [JavaScript, ECMAScript, XMLHttpRequest, SPA, Single Page Application]
---
So having written enough heavy AJAX applications I can tell you that a massive frustration is that there is
not any ability to detect natively when the browser follows a redirect over AJAX, this makes handling redirection difficult in the client.

Some might suggest that we should invest further into using web sockets or someting instead of AJAX. However using AJAX seems to fit better
with well established MVC web frameworks that are able to respond with JSON or XML.
This will allow us to copy over much of the existing sites functionality without making many changes on the server side.

Firstly I would like to describe some of the merits of being able to see where the browser has been sent to:

-    HTTP redirection is used for many reasons on the web, most frameworks would have to use special conditions to 
     manage conditional redirection based upon content type, locales and other rules
-    The Content-type the server responds with may differ from the response the client expected,
     this is likely to occur when the server can't respond to the Accepts header sent so sends HTML or similar instead.
     Without having redirection information the client application needs to validate the content returned before parsing.
-    Use of redirection to inform the AJAX application of a change in API
-    Redirecting the client application to relevant locale, encoding or device type
-    Having a simpler server side controller that responds with the same behaviour for all conditions is always a benefit

I'm having a difficulty in understanding when the feature came into browsers in the first place to auto follow redirection however
I can only assume it was put in to simplify the method usage.

So I did some further digging and managed to find that this feature was recommended to prevent redirection when specified but it was dropped
from the specification [in the W3C mailing list](http://lists.w3.org/Archives/Public/public-webapps/2010OctDec/0812.html).

It looks to me like followRedirects was instead replaced with doing the right thing when the request is a [CORS](http://www.w3.org/TR/XMLHttpRequest/#infrastructure-for-the-send\(\)-method) however the client side writer might
not be in full control of the server response within the same domain either.

AJAX requests that redirect currently look like this:

-    JavaScript request URL http://mydomain.com/page.
-    Server responds with 301, 302, 303, 307, or 308 and sets the Location header to http://mydomain.com/page2.
-    Browser auto sends a request to the URL of the Location header (http://mydomain.com/page2).
-    Server responds to request http://mydomain.com/page2.
-    Browser then completes the XMLHttpRequest returning the headers of the second request to http://mydomain.com/page2.

Compressing the history of the request like this makes it impossible for the application writer on the client side to know when the server
responded with a redirection without the server implementing:

-    Server doesn't respond with redirection status code
-    Server returns the path somehow in the response: JSON key, HTTP X-headers.

For me the most logical way to reduce the complexity of JavaScript client side applications is the ability to uniformly handle requests it
understands by loading different templates from the redirected responses. For requests it doesn't understand or are out of the application
scope the client could redirect using window.location.href. For this to happen reliably however we need either the ability
to see all the responses from the server if redirects happen or to be able to turn following redirections off.

So W3C please add back to the specification the ability to set followRedirects to false and/or the feature to view the redirection
requests followed in XMLHttpRequest.
