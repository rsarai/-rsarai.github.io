---
layout: post
title: Some words about microservices
categories: [software]
excerpt: ""
---

_Originally published on medium_

![Photo by [Guillaume
Bolduc](https://unsplash.com/photos/uBe2mknURG4?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)
on [Unsplash](https://unsplash.com/search/photos/containers?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)](https://cdn-images-1.medium.com/max/2560/1*ZtB3bSD2qOkRT3xJ_5gDQg.jpeg)

*This year I decided to work on a side project and these are part of my
notes about it.*

Recently I found myself in a situation when I was jumping in on a
project, while the core developer was jumping out of the project, a very
common thing on startups. After a soft onboarding (*no onboarding*),
they gave me access to the codebase, there were 8 repositories.
Microservices. This type of approach to application development has got
a lot of attention in the last years, there are a lot of blog posts on
the topic, a lot of talks about the benefits of microservices based
architecture.

I had never worked on a project with this architecture before. I had
never wanted to work with this before. So I decided to study a little
bit more about the topic, my bigger source was the book [Python
Microservices
Development](https://www.amazon.com/Python-Microservices-Development-deploy-microservices-ebook/dp/B01N7N7BU9), and also some other blog posts, and here are my
notes, my questions, my concerns, and my angry thoughts about it.

### Monolith

All my previous experiences were with Monolithic applications, these
applications can be described as:

> A monolith is a single program that runs in a single
> process. --- James Shore

Usually, this type of application has all the required components
included. Here's a more visual representation of this formal concept:

![<https://www.slashroot.in/difference-between-monolithic-and-microservices-based-architecture>](https://cdn-images-1.medium.com/max/800/0*pn-Ge_GKlhNombRB.png)

All the work is made for a single part of the application. There are a
lot of benefits of this type of application, for example: building a
good test coverage is easy, you can organize your code in a clean and
structured way inside the code base. Since all your data is into a
single database the development of the application is simplified.
Testing our application in an end to end matter is simple. The
deployment is also a no problem: we can tag the code base, build a
package, and run it somewhere, Heroku for example (I'm a big fan of
Heroku).

To be successful with a monolith, you need careful design to make sure
different parts of the program are isolated. Sometimes even if you try
things can still be really coupled together. And the more teams you
have, the more difficult it is to maintain the concerns separated.

### Microservices

So, as I said microservices is one the most hyped contents around
software development, let's start by going through its origins:

The origin of this architecture often mentions Service-Oriented
Architecture ([SOA](http://www.soa-manifesto.org/)), the core principle is the idea that you organize
applications into a discrete unit of functionality that can be accessed
remotely and acted upon and updated independently. Each unit in this
preceding definition is a self-contained service, which implements one
facet of a business, and provides its feature through some interface.
However, it is common to say that microservices are one specialization
of SOA.

This type of architecture is seen as a solution to the dependency on the
monolith.

> A microservice is a lightweight application, which provides a
> narrowed\
> list of features with a well-defined contract. It's a component with a
> single\
> responsibility, which can be developed and deployed independently.

![<https://www.edureka.co/blog/what-is-microservices/>](https://cdn-images-1.medium.com/max/800/0*9ISh5N6LtUVjsuw2.png)

At first glance, microservices seem like the perfect solution to all the
disadvantages of the monolith. Each microservice is a small,
self-contained program. Developed completely independently, with its own
repository, database, and tooling. Services are deployed and run
separately, so it's literally impossible for one team to inappropriately
touch the implementation details of another. Unfortunately,
microservices move the coordination responsibility from development to
operations. Instead of deploying and monitoring a single application,
ops has to deploy and monitor dozens or even hundreds of services.

On this part of my research, a lot of comparisons came up like:
"Monolith applications are unreliable, not fit for complex applications,
unscalable and many other things, and microservices is the solution to
all of these problems." But of course, let's get into that.

### Microservice benefits

While the microservices architecture looks more complicated than its
monolithic counterpart, its advantages are multiple:

-   Separation of concerns
-   Smaller projects to deal with
-   More scaling and deployment options

### Microservices pitfalls

You need to be aware of these main problems you might have to deal with
when coding microservices:

-   Illogical splitting
-   More network interactions
-   Data storing and sharing
-   Compatibility issues
-   Testing

### Monolith vs Microservices

One thing that stuck with me was the Illogical splitting. According to
the book [Python Microservices
Development](https://www.amazon.com/Python-Microservices-Development-deploy-microservices-ebook/dp/B01N7N7BU9):

> "The first issue of a microservice architecture is how it gets
> designed. There's no way a team can come up with the perfect
> microservice architecture in the first shot. The design needs to
> mature with some try-and-fail cycles. Adding and removing
> microservices can be more painful than refactoring a monolithic
> application."

You can mitigate this problem by avoiding splitting your app in
microservices if the split is not evident.

> Premature splitting is the root of all evil

This specific point stuck with me. Did previous developers take the
wrong road when they decided to build a microservices architecture for a
startup project since the beginning?

"...and we didn't even launch yet"

Luckily, there are a lot of posts written by far more experienced people
to bring a light on this discussion. A great source of study is the
[Martin Fowler](http://martinfowler.com) blog if you don't know Fowler you should, he's an
author, speaker, and loud-mouth on the design of enterprise software. I
wanna focus on two articles:
[MonolithFirst](https://martinfowler.com/bliki/MonolithFirst.html) and [Don't start with a
monolith](https://martinfowler.com/articles/dont-start-monolith.html)].

### Monolith First

Fowler stars this post with two statements that came from his experience
with software development:

1.  Almost all the successful microservice stories have started with a
    monolith that got too big and was broken up
2.  Almost all the cases where I've heard of a system that was built as
    a microservice system from scratch, it has ended up in serious
    trouble.

> You shouldn't start a new project with microservices, even if you're
> sure your application will be big enough to make it
> worthwhile. --- Martin Fowler

![](https://cdn-images-1.medium.com/max/800/0*FZmBW5KmEP3V6eP3.png)

The main reason for him is the classic Yagni (You Aren't Gonna Need It).
When you're building a product for a startup how sure are you that your
product is gonna stick? How sure are you that is gonna be a success? The
best way to find out is to build a simplistic version and see how well
it works.

### Don't start with a monolith

The full title is: "Don't start with a monolith ... when your goal is a
microservices architecture". This post was written by [Stefan
Tilkov](https://www.innoq.com/blog/st/), he starts with:

> I'm firmly convinced that starting with a monolith is usually exactly
> the wrong thing to do. --- Stefan Tilkov

He states that in the majority of cases, it will be extremely hard to
cut up an existing monolith system. This is coupled with the way you
develop your monolithic application if you are able to build a
well-structured monolith you have no need of microservices. However, the
reality is quite different:

![](https://cdn-images-1.medium.com/max/800/1*KqIf71SWpAZ_PBhQxJ7YXw.jpeg)

Microservices main benefit, in my view, is enabling parallel development
by establishing a hard-to-cross boundary between different parts of your
system. In theory, you don't need microservices for this if you simply
have the discipline to follow clear rules and establish clear boundaries
within your monolithic application.

> I strongly believe --- and experience from a number of our recent
> projects confirms this --- that when you start out, you should think
> about the subsystems you build, *and build them as independently of
> each other as possible*. --- Stefan Tilkov

...

In the end, the answer to the architectural decision is: https://twitter.com/KentBeck/status/596007846887628801
