---
layout: post
title: Functional programming in PHP
---

This evening I want to talk about why programming in a functional style using PHP is probably a bad idea. The three key components of functional programming and where PHP goes wrong:

## Higher-orderism

A fundamental part of functional programming is the ability to pass functions around as parameters to other functions for them to execute and retrieve the results of. For example a function like `array_filter` that takes an array and a function and returns an array of items from the original array that yielded a truthy result from the function passed in. Saying all of that is a real mouthful isn't it and yet the `array_filter`'s type signature gives you no advice on this, all it says is that it takes an array, a function and returns an array. Keeping in mind arrays are also hash maps you're really wondering what this could all be. Of course, it being a built function this isn't much of a problem because your IDE can help out and you can kind of guess if you've heard of a filter function before what this is going to do, but when it comes time to start incorporating higher-orderism in to your own code a whole lot of information is lost in the type signature. Now you're simply taking a closure and the parameters and return type of that is lost. Bugs become much more likely and also I find it a bit annoying to have to go to the documentation to find out what order the parameters are in for the closure.

The same applies to returning functions as well.

Another problem is the inability to pass functions in by name, I mean you can but the name must then be a string instead of just its name as a symbol. Also the horrific syntax for passing methods: `array_map([$this, 'apply'], $arr)`. Yikes.

## Lists

PHP doesn't have a list type, at least not one that's built in to the actually used areas and not some Spl something that nobody really uses. This makes accepting arrays risky business. Not only do you not have the ability to do things like set a limit on the size of an array you take or limit it to only containing one type of value, but you can't even guarantee you'll even get a simple list. Lists are a fundamental part of functional programming and the concept doesn't exist in PHP.

## Recursion

Without tail call optimization you're not really able to apply another primary principle in functional programming: recursion. Recursion on lists when you don't have lists and have a very rapidly growing memory footprint?

## Any other goodies?

Truth be told PHP simply doesn't have anything that helps make it more functional. Along with the above it lacks currying, generics, short closure syntax, pattern matching, macros and a strong community interest in these areas. Ultimately all of these things combined mean that if you're trying to apply functional principles in PHP beyond the odd `array_map` and stuff like that then you're writing code that will be less legible and more brittle than an object oriented equivalent.
