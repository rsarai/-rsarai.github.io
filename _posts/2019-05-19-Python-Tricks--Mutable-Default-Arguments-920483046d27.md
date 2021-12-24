---
layout: post
title: Python Tricks. Mutable Default Arguments
categories: [python]
excerpt: ""
---

_Originally published on medium_

TLDR; Avoid using an empty list on any mutable object as a default argument to
a function

Working with Python for almost two years now, and still learning new
things every day. Don't think it's something extremely advanced, don't
get me wrong sometimes it is, but not today. I wanna talk about
something much more simple like default arguments.

Default arguments are the default values set on the function definition
like:

```python
def function(element=1, other_element=0):
    pass
```

This function can be called in several ways:

-   function()
-   function(1000)
-   function(10, 20)

The Python documentation also adds an important warning:

> **Important warning:** The default value is evaluated only once. This
> makes a difference when the default is a mutable object such as a
> list, dictionary, or instances of most classes.

Translating this to code:

```python
def f(a, L=[]):
    L.append(a)
    return L

print(f(1))
print(f(2))
print(f(3))
```

This will print:

```python
[1]
[1, 2]
[1, 2, 3]
```

We can see that the default argument **L=\[\]** is only evaluated once,
so a new list is created once when the function is defined, and the same
list is used in each successive call. But you may think: "Hey, this is
wrong, I want to receive a list with each individual value" like the
following:

```python
[1]
[2]
[3]
```

To achieve this and don't allow the default argument to be shared
between subsequent calls, you can write the function like this instead:

```python
def f(a, L=None):
    if L is None:
        L = []
    L.append(a)
    return L
```

Avoid having mutable default arguments to functions, use **None**
instead.
[Here](https://nikos7am.com/posts/mutable-default-arguments/) is an awesome post about how to take advantage of
this Python feature.
