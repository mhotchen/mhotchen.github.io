---
layout: post
title: "Behavior-driven development for microservices"
---

## What is BDD?

[Behavior-Driven Development (BDD)](https://en.wikipedia.org/wiki/Behavior-driven_development) is a subset of [Test-Driven Development (TDD)](https://en.wikipedia.org/wiki/Test-driven_development) that focuses on verifying the behavior of a piece of software by using the terminology and language of the domain you're working in. Whilst TDD will often have elements of this, it usually isn't the focus. In some situations the 'best practice' for TDD means that you are verifying small individual pieces from a technical perspective, often missing out on testing whether the application behaves in the way the consumer/business wants.

BDD is often confused with integration tests but they aren't the same thing (although boundaries are almost always crossed). An integration test can go from the high level "I make this request and I expect this response" to a much lower level style of unit testing wherein you mock the dependencies and verify that the dependencies are used correctly (for example you mock a database repository and verify that the unit you're testing will call the right method with the right parameters to get the object it should be working with). Martin Fowler has [an excellent article on the second style](http://martinfowler.com/articles/mocksArentStubs.html).

BDD focuses on using language and terminology that closely aligns with the domain you're in (the business/industry) to make it easier to test based on what the business actually wants. Most tools will further narrow the focus to try and align the tests and the user story as closely as possible.

One specific subset of tools for BDD use a DSL called Gherkin (for example [Cucumber for Ruby](https://en.wikipedia.org/wiki/Cucumber_%28software%29)), which aims to be a more common language between the business and developers (much like user stories themselves).

## Applying BDD to microservices

In this article, I'll be focusing specifically on the Gherkin style BDD because because when applied to microservices it offers a few major wins:

* It helps define the boundaries of your microservices
* It gives developers a huge safety net
* It means that over time the microservice will continue to be aligned with the business needs

The fundamental problem with testing microservices using Gherkin is that their interface isn't something that directly relates to a user, and a single user story may cross the boundaries of several microservices. Let's create an example user story, and an example microservice architecture:

---

### Story

As a rich user I want to get a 5% off discount voucher when I spend more than $1,000.

#### Acceptance criteria

* If I spend up to and including $1,000 do I receive no vouchers?
* If I spend more than $1,000 do I receive a 5% off voucher to use on my next purchase?
* Is the voucher valid forever?
* Is the voucher valid immediately?

---

We'll assume that in our imaginary microservice architecture a voucher system already exists, but it requires the implementation of vouchers that never expire. We'll also assume that vouchers have never been generated at the end of an order before, so this now requires modification to both the order creation and the voucher creation microservices.

Immediately you can use Gherkin notation to verify the behavior for the platform the user is using with an almost one-to-one mapping to the user story's acceptance criteria.

For the other platforms, you will want to look at the subset of the story that they work with. The ordering platform should cover the entire user story in its Gherkin notation tests, but the voucher system would have Gherkin notation tests only relating to the concept of creating vouchers that are valid forever and valid immediately, as well as ensuring that existing tests cover the inverse situations properly.

Because you will often have BDD tests that can be shared with the business easily in the application putting the microservices together (in our case the one the user places the order through) you have a little more flexibility in your microservices to verify the behavior in a way that doesn't fit the natural language concept. This can make it tempting to use fixtures, but I would strongly suggest abstaining for a few reasons:

* The other developers will often struggle to work out exactly what component you're testing
* Instead of your tests breaking in isolation, helping to narrow down what you broke, a change will often break many unrelated tests, leaving you no wiser as to what broke
* It means your tests will often end up unrelated to what the business actually wants you to test

A better approach is to pay attention to the original story, and design your Gherkin notation around it as closely as possible. This usually means going beyond testing the request/response match a certain shape, but for example also verifying that the data is stored correctly or ensuring the response happened quickly enough. This means that your test suite—although more complex—follows the business strategy over time and ensures that you're actually writing your code in a way that aligns with business (and consumer) needs.

Fixtures are important since microservices have to adhere to a much stricter contract than a form viewed by humans, but the fixtures should be verified as part of a larger suite because the strict contract of a microservice is just one behavior that needs verified.
