---
layout: post
title: "Stop building REST APIs"
---

A popular meme in the construction of software is REST APIs. These aim to utilize the full power of HTTP in order to provide a better and easier to use API. In this article I'll try to explain why this is usually a bad idea and suggest how you should be designing your APIs instead.

## REST and HTTP

The term REST is coined by Roy Fielding in his PhD dissertation [Architectural Styles and the Design of Network-based Software Architectures](https://www.ics.uci.edu/~fielding/pubs/dissertation/top.htm). This dissertation is highly accessible and worth reading. It's effectively a retroactive analysis of what it would look like to design the World Wide Web, or something like it.

Some of the key takeaways include identifiable resources, statelessness, and discoverability through hyperlinks.

HTTP can be seen as an implementation of REST, although it came first. It provides a way of sending and modifying resources and metadata over TCP/IP using identifiers (URLs). For the most part it's stateless, and resources are discovered and shared through hyperlinks.

## REST and APIs

I want to preface what I'm about to say by emphasizing that this does not apply to every situation; there are a specific subset of APIs that should use REST such as those focused on the semantic web and JSON-LD. But I don't want to ruin the point by adding a clause for every unique situation out there. When I talk about APIs below I'm talking about those that are consumed by specific and unique implementations, rather than those with a more generic consumer such as the ones reading JSON-LD data. For example Stripe's API would be the sort of API I'm talking about here.

APIs have a very different style of interaction in comparison to what REST is designed for. REST is designed to be "driven by client selection of server-provided choices that are present in the received representations or implied by the user’s manipulation of those representations" ([REST APIs must be hypertext driven](http://roy.gbiv.com/untangled/2008/rest-apis-must-be-hypertext-driven)). APIs are driven by a documented structure that is known in advance to the API consumer. Whereas a REST resource can change fairly arbitrarily, even changing the location of resources freely, doing this in an API will break the consumer. In other words the resource names should be meaningless, but in APIs they're meaningful because they're tightly coupled to an expected behavior.

Resources within an API are not discovered through flexible hyperlinking. If APIs were designed this way it would make them largely useless to the consuming code. For example, let's propose that you design a root JSON document for your API that describes each resource available in your API with hyperlinks, much like a typical REST resource would:

{% highlight javascript %}
{
	"navigation": [
		{"Home": "/"},
		{"My account": "/users/foo@bar.com"},
		{"My cart": "/users/foo@bar.com/cart"},
		{"Cat toys": "/shop/category/cat-toys"},
		{"Cat food": "/shop/category/cat-food"}
	],
	"content": "Welcome to Cat Co."
}
{% endhighlight %}

These links are then subject to change, and the resources underneath them are subject to change as well. The consuming client would therefor have to be adaptable to said change, as well as provided a method for navigating the content arbitrarily. One might call this a browser, and the resource would by necessity become more of a markup language than a fairly rigid structure that allows the computer to do useful things behind the scenes. We already have a solid REST implementation that provides this style of navigation and it's called the World Wide Web.

For an API to be useful it has to be easy to consume and remain fairly rigid in its implementation to avoid breaking consumers. If the consumer has to build a browser, that's not easy to implement (nor does it work for consumers that aren't powered by humans, such as in a service oriented architecture). If it remains rigid and avoids change, then it isn't REST.

## What an API should be

An API should be rigid in its implementation to avoid breaking consumers. An API should be well documented and easy to consume with all error and edge cases specified and easy to handle. Hyperlinks could be provided to other resources where appropriate, such as when you've just created a resource and want to provide a link for retrieving it in the future, or for pagination, but the consumption of the API is *not* hyperlink-driven.

JSON over HTTP provides a way of implementing a good API. There are good specifications for designing such an API such as [json:api](http://jsonapi.org/), and good tools for building the documentation and making it easy for consumers to understand such as [swagger](http://swagger.io/). Many libraries and frameworks exist for both clients and servers for building and consuming such APIs rapidly and correctly. Although APIs built like this *aren't* REST they are useful.

So stop implementing REST APIs and start implementing something JSON over HTTP. There are many other perfectly valid choices as well such as [Apache Thrift](https://thrift.apache.org/) although JSON over HTTP is likely to be the easiest one to implement both on the server and as a consumer.
