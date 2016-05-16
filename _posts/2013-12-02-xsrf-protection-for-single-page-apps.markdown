---
layout: post
title: "XSRF protection for single page apps"
date: 2013-12-02 23:03
published: false
comments: true
categories: [XSRF, security, Single Page Application, SPA, ECMAScript, JavaScript]
---

Securing [Single Page Applications](http://en.wikipedia.org/wiki/Single-page_application) might seem daunting however lets start off with protecting against XSRF attacks.

So what exactly causes an XSRF exploit?

This happens when a an authenticated computer is tricked into calling something the user didn't want.
So for example: A user is on a message board which loads an iframe calling a payment on the users bank. The user is logged in so the payment is taken from the users bank account because of the cleverly constructed link.