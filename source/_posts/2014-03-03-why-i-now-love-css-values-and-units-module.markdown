---
layout: post
title: "Why I now love CSS Values and Units module"
date: 2014-03-03 21:29
comments: true
categories: 
---

## Intro

So when I first read about [CSS Values and Units module](http://www.w3.org/TR/css3-values) I completely dismissed it. 
At first I thought using the `calc()` property would be a mad move for CSS and how browsers would support new features.
My initial drive to thinking this was that we have JavaScript and it is a known use for calculating properties.
I don't really think JavaScript is the best use for producing layout but it seemed far better than adding essentially a dynamic language into a declarative format.
 
## The rise of the CSS framework
 
You only have to sift through some of some of the modern frameworks code to realise that there is a units problem at hand here.
The frameworks I have had a chance to work with all have had the issue of mixing units and having to constantly specify dimensions.
 
The problem I see is code like this:
 
```
  <style>
    body {font-size:12px;}
    .ems span {font-size:0.8em;}
  </style>

  <div class="ems">
    <span>Call me maybe</span>
 
    <br />
 
    <span><span>Call me maybe</span></span>
 
    <br />
 
    <span><span><span>Call me maybe</span></span></span>
  </div>
```
Which if you run in a browser you will notice the half arsed Carly Rae Jepsen lyrics fade away.

This can however be fixed by using [rem](http://www.w3.org/TR/css3-values/#rem-unit) units instead:
```
  <style>
    body {font-size:12px;}
    .rems span {font-size:0.8rem;}
  </style>
  <div class="rems">
    <span>Call me maybe</span>
 
    <br />
 
    <span><span>Call me maybe</span></span>
 
    <br />
 
    <span><span><span>Call me maybe</span></span></span>
  </div>
```

Which if you run in the browser you will see that it stays consistent size for all levels in the DOM.

So great news rem's actually solves the issue here, however in wades Internet Explorer 8 - which if you are required to support doesn't work with rem units.

So we are stuck with either using px's or ems, which both can be hard to maintain for the reason above.
 
## Introducing Myth
 
So I could tell you lots about preprocessing languages like LESS and SASS, however I have always used them in angst.
 
[Myth](http://myth.io) however is basically a mix of all the latest [W3C](http://www.w3.org) specifications which means once all browsers support the features the amount of processing can be decreased over time.

This is where `calc()` and CSS variables come into play with the ability now for us to use an absolute size to reference element and font sizes.
Over time we can then change these to rem units with ease allowing for a much more flexible layout.

So I created a quick [demo site for in browser myth compiling](http://jonathankingston.github.io/mythhub/) so I could show examples.

```
:root {
  var-base-font: 12px;
}

h1 {
  font-size: calc(var(base-font) * 2);
}

h2 {
  font-size: calc(var(base-font) * 1.75);
}

h3 {
  font-size: calc(var(base-font) * 1.5);
}
```
[Example CSS Myth font calculation](http://jonathankingston.github.io/mythhub/?m=%3Aroot+%7B%0D%0A++var-base-font%3A+12px%3B%0D%0A%7D%0D%0A%0D%0Ah1+%7B%0D%0A++font-size%3A+calc%28var%28base-font%29+*+2%29%3B%0D%0A%7D%0D%0A%0D%0Ah2+%7B%0D%0A++font-size%3A+calc%28var%28base-font%29+*+1.75%29%3B%0D%0A%7D%0D%0A%0D%0Ah3+%7B%0D%0A++font-size%3A+calc%28var%28base-font%29+*+1.5%29%3B%0D%0A%7D)


When browser support is sufficiently high enough for the application you are building, the variables can be changed to give the exact same output using rem's rather than px.
```
:root {
  var-base-font: 0.75rem;
}

h1 {
  font-size: calc(var(base-font) * 2);
}

h2 {
  font-size: calc(var(base-font) * 1.75);
}

h3 {
  font-size: calc(var(base-font) * 1.5);
}
```
[Example CSS Myth font recalculation](http://jonathankingston.github.io/mythhub/?m=%3Aroot+%7B%0D%0A++var-base-font%3A+0.75rem%3B%0D%0A%7D%0D%0A%0D%0Ah1+%7B%0D%0A++font-size%3A+calc%28var%28base-font%29+*+2%29%3B%0D%0A%7D%0D%0A%0D%0Ah2+%7B%0D%0A++font-size%3A+calc%28var%28base-font%29+*+1.75%29%3B%0D%0A%7D%0D%0A%0D%0Ah3+%7B%0D%0A++font-size%3A+calc%28var%28base-font%29+*+1.5%29%3B%0D%0A%7D)

So Myth is in its early stages but even so its clear this style of CSS development could be the most future proof.

## Future developments for Myth

I would like to see the following:

-    Would be great if I could configure a per browser output
     -    Selective browser extensions, currently it supports browser auto-prefixing like: `-webkit-transition: opacity 1s;` output instead just the corresponding browser specified
     -    Provide the least amount of fixes for modern browsers
-    Choose to auto follow @include and compile all files into one .css file
-    Output source maps
-    Allow to declare what screen size the user is to strip out media queries that are not relevant


## Future developments of CSS values specification

I would like to see the following:

-   [Less ambiguity in value precision](http://www.w3.org/TR/css3-values/#numeric-types) or the ability to provide a precision
    - I think this matters more when you are using multiple compilers that you get a consistent numeric precision.
-   WEBIDL format or similar to improve parsers and validators?