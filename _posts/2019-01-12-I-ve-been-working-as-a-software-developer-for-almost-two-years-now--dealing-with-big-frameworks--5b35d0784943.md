---
layout: post
title: What's up with Gunicorn
categories: [python]
excerpt: ""
---

_Originally published on medium_


I've been working as a software developer for almost two years now,
dealing with big frameworks like Django and React (not actually a
framework, but let's go with that). These web frameworks come with a set
of coding components and a set of basic structuring tools that help
simplify web development. This week I realize that although these tools
provide me a great advantage on the coding side, they also make me lost
a little of perspective about what was going on underneath the surface.
To get some perspective back, this week I decided to study some basic
things related to web development.

### Gunicorn

Gunicorn is a shortened for Green Unicorn.

![Gunicorn](https://cdn-images-1.medium.com/max/800/0*0SBG8hrOhT9tH5lk.jpg)

It's one of many Web Server Gateway Interface (WSGI) implementations and
it's commonly used to run Python web applications. This particular
implementation is important because it is a stable, commonly-used part
of web app deployments and is used by some big Python applications
around the world, such as
[Instagram](http://instagram-engineering.tumblr.com/post/13649370142/what-powers-instagram-hundreds-of-instances).

### WSGI

On the practical side, WSGI is a specification that describes how a web
server communicates with web applications. However, in the old days,
when Python was created other patterns were used. Everything started
with Gregory Trubetskoy trying to [get rid of the chains of Visual
Basic](https://grisha.org/blog/2013/10/25/mod-python-the-long-story/) to add Python to his project, he was able to embed
the Python interpreter on an Apache web server and called mod_python.
This method worked for a few years, but summing up the fact that it was
not a standard specification, security vulnerabilities were discovered
and the development stalled it was necessary for the community to
develop a consistent way to execute Python code for web applications.

Therefore in December of 2003, the [Python Web Server Gateway Interface
v1.0](https://www.python.org/dev/peps/pep-0333/) was created to propose a standard interface between
web servers and Python web applications or frameworks, to promote web
application portability across a variety of web servers. The goal was to
imitate one specific behavior of Java. Like Python, Java has a wide
variety of web application frameworks available in order to make these
different frameworks to run in any server, the Java's "servlet" API was
implemented. Resulting in any applications written with any Java web
application framework is able to run in any web server that supports the
servlet API.

The original PEP 333 also says:

> ...Thus, WSGI **must** be easy to implement... Thus, simplicity of
> implementation on *both* the server and framework sides of the
> interface is absolutely critical to the utility of the WSGI interface,
> and is therefore the principal criterion for any design decisions.

Focusing on simplicity the specification has two sides: the "server" or
"gateway" side, and the "application" or "framework" side.

![[Full
Stack Python](https://www.fullstackpython.com/wsgi-servers.html)](https://cdn-images-1.medium.com/max/800/0*LTkOmyVqRXO8OiH6.png)

The server-side invokes a callable object that is provided by the
application side. The specifics of how that object is provided are up to
the server or gateway. A nice thing that's worth mentioning is that the
term "callable" refers to an object with a \_\_call\_\_ method if you
never heard about this a good source of information about the subject is
the book Fluent Python.

Django's primary deployment platform is
[WSGI](http://www.wsgi.org/){.markup--anchor .markup--p-anchor}. Again,
the key concept is the callable object, it's commonly provided as an
object named `application`{.markup--code .markup--p-code}in a Python
module accessible to the server, usually: **\<**project_name\>/wsgi.py.

```python
"""
WSGI config for boilerplate project.
It exposes the WSGI callable as a module-level variable named ``application``.
For more information on this file, see
https://docs.djangoproject.com/en/1.9/howto/deployment/wsgi/
"""
import os
from django.core.wsgi import get_wsgi_application
os.environ.setdefault("DJANGO_SETTINGS_MODULE", "project_name.settings")
application = get_wsgi_application()
```

This file exposes an **application** variable so that a WSGI server can
use the **application** as a hook for running the web app.

![[Full
Stack Python](https://www.fullstackpython.com/wsgi-servers.html)](https://cdn-images-1.medium.com/max/800/0*jbQqi_9FLZV8p4AM.png)

### Gunicorn Architecture

Back to gunicorn. Gunicorn implements the PEP 3333, therefore it is able
to run Python web applications that implement the application interface.
For example, if you write a web application with a web framework such as
Django, Flask or Bottle, then your application implements the WSGI
specification.

The Gunicorn has a simple architecture, based on the pre-fork worker
model. This means that there is a central master process that manages a
set of worker processes. The master is a simple loop that listens for
various process signals and reacts accordingly, it never knows anything
about individual clients, but it manages the list of running workers by
listening for signals to increase or decrease the number of running
workers. All requests and responses are handled completely by worker
processes, this is the reason for the **pre** in **pre-fork,** the
processes are forked before a request comes in. One more thing, the
workers can be synchronous or asynchronous, and are very efficient only
needing 4--12 worker processes to handle hundreds or thousands of
requests per second. About the number of workers the Gunicorn
documentation says the following:

> Always remember, there is such a thing as too many workers. After a
> point your worker processes will start thrashing system resources
> decreasing the throughput of the entire system.

### Running Gunicorn

When Gunicorn is installed, a **gunicorn** command is available which
starts the Gunicorn server process.

So for a typical Django project, invoking gunicorn would look like:

```bash
gunicorn myproject.wsgi
```

This will start one process running one thread listening on
**127.0.0.1:8000**. On my project, I use gunicorn to start up our web
dynos on Heroku, the configurations are in a file called **Procfile**:

```bash
web: gunicorn {{project_name}}.wsgi --limit-request-line 8188 --log-file -
```
