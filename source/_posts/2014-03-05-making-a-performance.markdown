---
layout: post
title: "Making a performance"
date: 2014-03-05 21:15
comments: true
categories: 
---

## Don't over optimise too early
 
A common trend I see more in JavaScript, than in any other language is to over optimise every line of code.
This comes in many flavours but the two most common are:
-    Optimising to the least lines of code
-    Optimising code to be most performant
 
## Performant code

The scope of the project is really important here: if we were making an in-browser game then there may be justification for getting the most optimal code for the browser.
 
Most websites, **do not** require the level of interaction that developers expect to warrant optimising for the computer.
 
Having code that can be diagnosed easily in smaller isolated components, makes it much easier to improve performance in a much safer manner, when the performance is needed.

Over optimising early looks a lot like this:
 
```
  var chosenPost = false;
  var findPost = function (post, title) {
    if (chosenPost !== false) {
      return chosenPost;
    }
 
    var i = posts.length;
    var post;
    while (i--) {
      post = posts[i];
      if ('title' in post && post.title === title) {
        chosenPost = post;
        return chosenPost;
      }
    }
 
    return false;
  };
 
  var posts = [
    {title: 'Optimisation scandal'},
    {title: 'School reunion'},
    {title: 'Something else'},
  ];
  var chosenPost = findPost(posts, 'Optimisation scandal');
```

In the example above, the coder has attempted to improve performance by making the method not search through the posts the second time round.
Firstly, this code increases application risk due to two factors:
-    Introducing while loops of this nature are poorly understood, the [off by one error](http://en.wikipedia.org/wiki/Off-by-one_error) happens here easily.
-    Adding in caching introduces other risks:
     -    It is easy for developers to skip over these lines of code
     -    Cache rules can become sullied with lots of conditionals
     -    This stops the method being [indempotent](http://en.wikipedia.org/wiki/Idempotence): developers can easily mess with the global variable and cause issues.

What is also worth commenting on here - is actually, these lines of code [do not always](http://jsperf.com/loops/145) give the performance improvement developers expect.
Until the need for an optimisation has been proven and the performance improvement has also been proved, I would always leave them out, unless the coding style expects them.

The questions I try to ask myself when writing an optimisation is:
-    How long will it take to implement?
-    How hard does it make the code to read?
-    How much time will is save the user?
-    Do the users need that saving of time or do I?

## Lines of code
 
Optimising for the least number of lines of code is mainly a misconception that programmers are at their most optimal when they have to write the least lines.

Write speed for a programmer is a factor, however it is not the only one.

For teams working together on the same code base, comprehension speed, I would argue, is actually the most important. 

When a programmer starts at your company, whether they are junior or senior, it is likely there will be a lot of training for the new starter either by themselves, or with another programmer.
For every simplification to a programmers write speed comes an increase in comprehension and training required.

Reducing key presses leads to code that is harder to maintain and also comprehend:
 
```
  var thing,
      thing2,
      thing3,
      thing4 = true;
  thingAssign(thing, thing2, thing3, thing4)
  if (thing && thing2 && thing3) $('something').each(function () { $(this).addClass('selected')  })
  if (thing4) doSomething('thing')
```
 
The problems I expect new programmers to encounter with the above code are:
-    They may be unsure regarding what thing - thing4 get declared as
-    Difficulty understanding where [ASI](http://ecma262-5.com/ELS5_Section_7.htm#Section_7.9.1)(Automatic Semicolon Insertion) rules come into play
-    Comprehension of where if statements terminate
-    What is checked in the if statement `thing && thing2 && thing3`
     -    If either of the three variables is set to: false, null, undefined, NaN, 0, or '' then this test would fail
-    Remembering to insert brackets or commas after each statement in the if once another statement is required


## Clear coding standards help

One of the main reasons I have become so interested in [ESLint](http://eslint.org/), is because it will allow for strict coding standards that are not interpreted differently.

Personally, I would prefer to codify a set of coding standards as much as possible. This is mostly down to experience of seeing so many developers not follow standards.

## Summary

This was just a brief look into making development simpler for teams to manage. Most of this article is fairly basic but it is worth planning a set of coding standards and a clear coding strategy to create a clear company wide approach. This uniformity in the team should improve code output and limit mistakes.
