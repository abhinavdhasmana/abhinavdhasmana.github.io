---
layout: post
title:  "Welcome to Javascript and 'this' keyword"
categories: [JavaScript, ES6]
summary: Introduction to JavaScript 'this' keyword
author: Abhinav Dhasmana
---

_This is a part four of a series of blog where I am planning to capture on how we provide technical apprenticeship to new college graduates that intern with us. Here is_ [_Part 1_]({% post_url 2017-02-15-Git-101-and-its-day-to-day-usage-Part-1 %})_,_ [_Part 2_]({% post_url 2017-02-22-Git-101-and-its-day-to-day-usage-Part-2 %}),_ [_Part 3_]({% post_url 2017-03-24-Guiding-principals %})


* * *

Once we have the basic Git and our guiding principles in place, we start with actual programming. First step, install `nvm` and `node` from [here](https://github.com/creationix/nvm).

As mentioned before, we try to use as much open source as we can. We again turn to node school and solve these

*   [Javascripting](https://github.com/workshopper/javascripting)
*   [Scope Chain Closures](https://github.com/workshopper/scope-chains-closures)
*   [Introduction to ES6](https://github.com/domenic/count-to-6)
*   [Functional Javascript](https://github.com/timoxley/functional-javascript-workshop)

One of the thing that we needed more depth was on `**this**` keyword. So here are some of the examples we used to learn more about it. All the examples here would be written in `strict mode`


{% gist 6dc133b9a21be48401d6e29c5b114513 %}

This one is simple. When in `strict mode` there is no global context and you get `**this**` as `**undefined**`. If we remove line 1, we would get a `window` object if you run this in browser or the `global` object if you are run this in node.

Lets enhance this example

{% gist 209fb3ce9e93e8686b008f8310a05d78 %}

If you are surprised by seeing `{}/window object`as an output, remember that there will always be one`this` object whose context is node/window.

Lets look at nested function

{% gist f93f474d3c51424e046d9f0eec99666e %}

We do not expect any change in the value of `this` as the context of `insideFunction` is `whoAmI` function.

What if we actually want to use `this` inside `whoAmI`? `apply` to rescue

{% gist 2a0737eb808be7193994d27c09b6ceb8 %}


With `apply`, the context is passed and hence we can see the output we see at line 4\. What about line 6? This is still just a function invocation and not with any object, so not a method invocation. Hence `this` is still undefined.

Lets extend this further, and lets say we want to have `insideFunction` to have access to `this` object as well. We have two option: Option 1 is to use `apply` again.

{% gist d5e19fea6b0ce5abfc9be2889d040d50 %}

Option 2 is where we add `insideFunction` to `this`

{% gist 0b18d32035e58d055caee576f5c8e79d %}

Note that in the second scenario, we have eventually added `insideFunction` to `this` which means that it is now exposed to outside world and probably defeating the purpose of creating a nested function. I just wanted to show the approach.

Lets start with more complicated examples and move from function invocation to methods invocation.

{% gist 3c27bb33a22f37c650e554482ea91527 %}

The output is a `testObj` because the function `whoamI` is called on the `testObj` at line 10\. Just to make the above point even more clear, look at the example below

{% gist 294296cad88f4381f331aefd9da40e2a %}

The value of `a` changes as we are operating on the same object.

Lets add some actual use case scenarios of node where async is more common

{% gist 4f47239c8d24631af498f3dc802c2c7d %}

If you are surprised by the result that the second log statement does not print the `testObj`, its because what gets passed to setTimeout is not the `testObj` but just the function. Later, the function is called with the context in which it is running.

How do we fix this? `bind` to the rescue.

{% gist 644c124280b846a0813878ebeb63fd22 %}

`bind` will make sure that this whenever this function is called, its called with the context passed as a parameter.

Lets look at some examples with classes

{% gist 06990da9af9607d1e76d32fe81f76564 %}

I am leaving arrow functions for another post.

Are there some important scenarios that you encounter/ stumped you on regular basis? Let me know and I’ll add it.

All the examples are in this [github repo](https://github.com/abhinavdhasmana/javacriptThisKeywordExamples).