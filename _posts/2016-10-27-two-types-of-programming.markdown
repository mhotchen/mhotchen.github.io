---
layout: post
title: "Two types of programming"
---

You can split most things in the world in to two categories, for example people that like chocolate and people that have no reason to live. In software there are many such divisions but what I'm presenting in this essay isn't in fact one of them. It is however a categorization of what I believe to be the two most popular and influential types of programming (and some of the gray matter in between).

So the two types are: solving human problems, and solving computational problems. Human problems will be the one that users are most familiar with. It's about automation of a human task, for example booking airline tickets or online banking. Computational problems however are where you're focused on having the computer, well, compute something. For example compression.

The scale isn't black and white, but by having this separation you can begin to understand why you have the Linus' of the world who shun anything except pure C, and at the same time the Rich Kickey's who believe in abstractions, DSLs, and to an extent not really caring about how the computer actually achieves what you wanted (by using something like the JVM). By understanding this separation of problems you are also better prepared for tackling a problem with the right tools.

At the one extreme where you're encoding enterprise specifications in to software you have your Scalas and your Clojures which allow you to create specific DSLs for the situation, along with creating higher level abstractions. You would rarely care about how these DSLs are computed and parsed, instead focusing on creating the right language for you and the business to understand each other. It could be argued that declarative languages such as SQL would sit in a more extreme position, but these are often used in conjunction with other tools to achieve the balanced requirements of business desires and computational reality.

Making your way in to somewhat caring about computation with Java and C# where you can't really build DSLs but you have a strong abstraction over the computer along with tools to create powerful abstractions of your own. These are your classic enterprise level languages because businesses rarely need to worry about what the computer is doing, they simply want the result and the ability for the software to be easily maintained. I would predict that the Very High Level languages will supersede these soon, such as the ones I gave above.

Further down the rabbit hole in to that gray area of very general purpose with python, PHP and to a lesser-extent C++ (which I'll discuss in a moment). These don't have the same power as the above languages in terms of abstractions, but from them you're able to drop down on to the computer you're running the software on and can be precise at moments in how your software runs. C++ makes that a lot easier than the other two, but modern C++ also allows for some very powerful abstractions. I personally find the language to be so cumbersome in the basics that I haven't spent enough time to really appreciate how it brings the power of C for computation in to a world where you can build up abstractions.

Finally we're down in to the nitty gritty with C, Rust, and a bit of hand rolled assembly if you have a long gray beard. These would be useful if you're building a filesystem, web server, or a similar problem where the computer's exact actions are important.

I don't think that programmers need to know about both extremes in depth, or even care; if preferred they should look at where they sit on the scale and focus on that. As is reality, you would imagine that most people will work somewhere between the middle and the extreme enterprise specification stuff. I've worked with many people who care more about one side than the other, and whilst debates can get a little heated over the right way of doing things, it's a complimentary trait if it can be recognized and honed.
