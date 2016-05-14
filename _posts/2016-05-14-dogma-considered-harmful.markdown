---
layout: post
title: "Dogma considered harmful"
---

In software development there is many methodologies and guidelines to follow that in theory (and often in practice) help you develop better software. Unfortunately I've encountered many developers with a very rigid mindset who will follow such methodologies and guidelines to the detriment of the software they're building. By following a certain guideline or methodology dogmatically they may reduce the software quality, the test quality, or their/their team's productivity.

Software developers should have a mindset of being like water: you arrive in to a situation and you adjust yourself and your tools to your surroundings. Some people will read that and say that it means you'll never make improvements, but that's the exact mindset I'm talking about: you're dogmatically applying what I write. Adjust yourself to the situation, understand it and its warts, and work with your team to build a shared solution that most people are satisfied with.

One example might be forgoing integration tests because they don't fit within the BDD or TDD methodologies used. A second might be trying to apply scrum within a company that isn't condusive to it. A third might be applying full HTTP/REST to APIs, to the point where consumers have to make many additional HTTP requests to build up a representation that's actually useful.

It's a good idea to have an understanding of what a dogmatic approach to applying whatever it is you want to apply can achieve, but it's also worth knowing what the trade-offs are and whether those trade-offs are important to you. Using the HTTP/REST example, if your API is for a mobile app and it's having to make dozens of requests to build a view for the user then you're going to eat the user's data, and also have performance issues. It's sometimes better have a larger and more complex resource that allows consumers to interact with it in a single request. By not dogmatically following REST you're increasing the quality of your software. There are examples of this with every guideline or methodology you pick up. Start understanding the weaknesses and adjust these guidelines and methodologies to work for your situation.
