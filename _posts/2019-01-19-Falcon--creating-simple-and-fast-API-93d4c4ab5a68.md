---
layout: post
title: Falcon. Creating simple and fast API
categories: [python]
excerpt: ""
---

_Originally published on medium_

![Photo by [Guillaume
Jaillet](https://unsplash.com/photos/Nl-GCtizDHg?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)
on [Unsplash](https://unsplash.com/search/photos/fast?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)](https://cdn-images-1.medium.com/max/2560/1*MattM04QSySowzXrPSO1dQ.jpeg)

Back to my studies, this week I decided to get away from the good the
well-defined world of the Python framework called Django and try some
different tools. This week I found
[Falcon](https://falconframework.org/).

Falcon, it's a bare-metal Python web API framework for building very
fast app backends and microservices with a particular emphasis on low
latency. The goal of the maintainers was to build something fast, close
connected with the network layer with a non-opinionated approach
resulting in a flexible codebase that could be used to build upon and
reuse over and over. Like a framework for frameworks.

> Why is this a thing?
> \- You

Well, I've been working with big frameworks for a while to have an
answer for that.

Big frameworks are great! They give you a starting point, a base
implementation to build your project upon. A lot of frameworks have a
very opinionated way of approaching things, which is both good and bad.
Of course, it depends on either your own personal opinions or the
problem you're trying to solve, but in this case, let's assume that
things start to get messy when you want to do something that's slightly
different from what the framework proposes. Usually what you will have
to do is to override a bunch of methods and implement this new, but very
similar behavior, on the places defined by the framework. Most of the
times, this is ok, but let's assume it's not. This time you had to
re-write a bunch of methods, that had nothing to do with your original
modification only because that's the way that the framework process
things.

Cases like this, are cases when you want to use simple and
non-opinionated tools. Let's get to code.

I want to build an API that exposes movie information. Typically I start
thinking about the resource, in this specific case: Movie. Falcon
borrows this concept from REST architectural style, resources are simply
all the things on the API that can be accessed by a URL. A resource on
falcon is represented by a simple python class that uses duck-typing, so
you don't need to inherit from any sort of special base class. In
practice, these classes act as controllers in your application. They
convert an incoming request into one or more internal actions and then
compose a response back to the client based on the results of those
actions.

The Movie resource above defines a single method,
`on_get()`. For any HTTP method you want
your resource to support, simply add an `on_*()`{.markup--code
.markup--p-code} method to the class, where `*`{.markup--code
.markup--p-code} is any one of the standard HTTP methods, lowercased
(e.g., `on_get()`,
`on_put()`,`on_head()`, etc.). These well-known methods are called
"responders". Each responder takes (at least) two params, one
representing the HTTP request, and one representing the HTTP response to
that request. By convention, these are called `req` and `resp`,
respectively. Now, open `app.py`:

This code creates a WSGI application and aliases it as
`api`. To run the API, make sure all the
files are in a *project_name* folder and:

```bash
$ pip install gunicorn
$ gunicorn --reload project_name.app
```

Now, when a request comes in for `/movies`, Falcon will call the responder on the images resource
that corresponds to the requested HTTP method.

![](https://cdn-images-1.medium.com/max/800/1*ZnGNdbIiyr9e-l0gCqQaxw.png)

The simplicity of this library it's the main advantage, with two files we have a fully functional API.
