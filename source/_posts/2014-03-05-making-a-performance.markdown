---
layout: post
title: "Making a performance"
date: 2014-03-05 21:15
comments: true
published: false
categories: 
---

## Don't over optimise to early
 
A common trend I see more in JavaScript than in any other language is to over optimise every line of code.
This comes in two flavours:
-    Optimising to the most least lines of code
-    Optimising to code to be most performant
 
## Performant code
 
Scope of the project is really important here, if we were making an in-browser game then there may be justification to getting the most optimal code for the browser.
 
Most websites, **do not** require the level of interaction that developers expect to warrant optimising for the computer.
 
Having code that can be diagnosed easier is smaller isolated components is much easier to improve performance when it is needed in a much safer manner.
 
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
In the example above, the coder as attempted to improve performance by making the method not search through the posts the second time round.
Firstly this code increases application risk for two factors:
-    Introducing while loops in this nature are poorly understood, the [off by one error](http://en.wikipedia.org/wiki/Off-by-one_error) happens here easily.
-    Adding in caching is another introduced risk:
     -    it's easy to skip over these lines of code
     -    Cache rules can become sullied with lots of conditionals
     -    This stops the method being [indempotent](http://en.wikipedia.org/wiki/Idempotence), developers can easily mess with the global variable and cause issues.
 
What is also worth commenting on here is actually these lines of code [don't always](http://jsperf.com/loops/145) give the performance improvement developers expect.
Until proven the need for an optimisation and proven the performance improvement I would always leave them out, unless the coding style expects them.
 
The questions I try to ask myself when writing an optimisation is:
-    How long will it take to implement?
-    How hard does it make the code to read?
-    How much time will the user save?
-    Do the users need that saving of time or do I?
 
## Lines of code
 
Optimising for the least number of lines of code mainly is a misconception that programmers are most optimal when they have to write the least lines.
 
Write speed, for a programmer is a factor, however it isn't the only one.
 
For teams working together on the same code base, comprehension speed I would argue, is actually the most important. 
 
When a programmer starts at your company, junior or senior. It is likely there will be a lot of training for the new starter either for themselves, or with another programmer.
 
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
 
The problems I expect new programmers for the above code are:
-    They may be unsure regarding what thing - thing4 get declared as
-    Difficulty understanding where [ASI](http://ecma262-5.com/ELS5_Section_7.htm#Section_7.9.1)(Automatic Semicolon Insertion) rules come into play
-    Comprehension of where if statements terminate
-    What is checked in the if statement `thing && thing2 && thing3`
     -    If either of the three variables is set to: false, null, undefined, NaN, 0, or '' then this test would fail
-    Remembering to insert brackets or commas after each statement in the if once another statement is required
