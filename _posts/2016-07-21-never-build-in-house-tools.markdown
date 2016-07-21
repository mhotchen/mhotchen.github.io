---
layout: post
title: "Never* build in-house tools"
---

Let me describe a mistake I've made in my career. A few years back I was brought on to a new project that was running behind schedule by a few months. The project wasn't a massive undertaking, in fact it was small enough to be written by a single developer up until I was brought on board. This developer had spent 9-10 months on the project so far. The project was a RESTful API, primarily doing CRUD with authentication, a combination that should allow most of the non-business logic to be handled by existing libraries and frameworks.

To my surprise the project had no framework to speak of and a lot of customized tooling. It was using a library for routing, but on top of it was a large class with lots of added logic, a custom way of handling parameters, embedded business rules, authentication, and custom dispatching. It was using a popular unit testing framework but with a custom framework built on top of it to instead use it as a functional testing framework. The JSON format chosen wasn't based on any existing format such as jsonapi.org or JSON-LD, but was a custom written set of rules. The list went on like this. Naturally this meant I had a surprisingly steep learning curve for such a small project. The difficulty of change was high as well as I struggled to make even small changes without having to change large swathes of the codebase.

At the time I didn't think much of it; the codebase was small and it did the task at hand. It still had some OOP elements to it, and a testing suite so that bugs were caught early. Over time I grew frustrated and undertook a refactoring effort, separating the routing, dispatching, authentication, validation, etc. in to their own reusable components. Unfortunately&mdash;like a virus&mdash;this actually spread. The reusable components became their own internal package, and new projects were using them as their primary framework. At the time of them being picked up they allowed quicker development due to familiarity and the fact that they were engineered specifically for that company's workflow, but over time the problems became apparent.

Sometimes it may cost less upfront to build the tool yourself. It was far easier to create a framework suited to the project's structure than to rewrite the project to use an existing framework. I couldn't measure by how much, but my guess would have been that it was 3-4x quicker for me to write a framework for the project than to use an existing one. So I maybe saved two months of development time. I should pat myself on the back, right? Then we saved some more time by using the framework to get another project up and running quite quickly since everyone working on this new project had used the framework, and it was a similar project. Good job, right?

The problem with long-term effects is that they're hard to measure quantitatively. Still, I can assert that the long-term effect of that decision was incredibly negative. Here are some of the costs and problems we encountered in the years that followed.

## Training new people

The first problem is that no matter how good the documentation is, firstly no-one reads it, and secondly no-one is familiar with your in-house tool prior to joining (unless you're rehiring someone). Developers are mostly smart and can learn a new language/framework/library in weeks, hours or minutes, but when a developer has already put thousands of hours in to one language and some existing frameworks then they're coming on with a much shorter learning curve and can begin focusing on learning the business domain far quicker. They will also never have the resources available for learning. If you hire someone who isn't familiar with your exact set of tools, they're going to have an easier time learning if the tools are popular.

## Hiring new people/team satisfaction

People are scared. They want to have known and popular tools on their CV to be able to find related work in the future. When you tell them it's something custom and everything they learn about it will be lost in the future, it's off-putting. Skills are transferable of course, but as a hiring manager given two equal candidates with one difference: one has used a custom framework that you don't know the quality of nor which practices it teaches the developer, and a second that's used a popular open source framework which demonstrates many best practices known to your particular discipline of programming, which do you prefer?

## Focus on the wrong things

Because you now have a tool to maintain, development time is removed from core business needs. This tool will never make money, and even if it stays closed source it has a maintenance cost. Open sourcing it will significantly increase that cost, and chances are your tool wouldn't be adopted by anyone without it being something truly unique and innovative. Open sourcing the tool *could* work as something to help alleviate the problems above, but chances are it won't. It will only increase the difficulty in maintaining the project. When using popular tools there are entire teams working full-time to maintain them. If the tool is free then there will usually be a company behind it with the core business value being aligned closely with the quality of the tool (eg. consultancies, hosting services, enterprise editions). If it isn't a free product then the tool is a direct product to sell for the business (Microsoft, Atlassian, etc.).

## Lack of maturity

Popular tools have usually had tens of thousands of man hours poured in to them before you've adopted using them. For the people who have put those hours in, this problem has become their specific domain of knowledge and they know more about it than you do. You might build a tool that solves your problem perfectly, better than an open source project would have, but then the requirements change, or new features are needed, and your tool doesn't adapt to these changes. You might have to make major releases of it quite often in order to keep up. Yet the popular open source tools have already gone through this quick and difficult cycle in their formative years as they began to gain widespread adoption.

This lack of maturity is true for the developers involved as well. They're now having to learn about a new set of problems, usually the hard way.

## Don't do it

There may be times in your career where it looks like the best way to do something is to build it yourself. This is rarely the right choice so be skeptical of your own mind. Research the existing tools. Actually try and use one first. If it doesn't quite fit, maybe you would be better contributing to it, or building a plugin for it, or something similar. Actually see how difficult that would be, speak to the maintainers about your problem. Building it yourself should only happen when all other ideas are exhausted.

*There are exceptions of course, such as Amazon's innovations in datacentres and cloud computing.
