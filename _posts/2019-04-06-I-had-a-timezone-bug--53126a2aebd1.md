---
layout: post
title: I had a timezone bug... again
categories: [python]
excerpt: ""
---

_Originally published on medium_

![Photo by [Luis
Cortes](https://unsplash.com/photos/QrPDA15pRkM?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/search/photos/time-zone?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)](https://cdn-images-1.medium.com/max/2560/1*wgQtuZPc_6Ay9CqzqpISLQ.jpeg)

When you work with time in a software product, timezone bugs *will*
appear once in a while. Sometimes you forget to consider a DST change,
sometimes you consider it wrong, and so on. I realized that even though
the same error doesn't happen twice, the previously solved errors give
the foundation to easily identify new ones. So, to not keep trusting my
memory here's the history of my timezone bug.

### The application structure

-   Python in the form of Django framework, as your backend
-   Postgres, as your database
-   And some javascript framework as your frontend.

The frontend is not important here, all the bug context was on the
backend.

### The general scenario

Let's assume we have a User abstraction--- like the following
model --- and we wanna show on a graph the number of *new members* *per
day* on the *last month*.

```python
from django.db import models
class User():
    email = models.EmailField()
    member_since = models.DateTimeField()
    first_name = models.CharField(max_length=255)
    last_name = models.CharField(max_length=255)
```

Simple right?

### The bug scenario

To pull the data and fill in the graph we made a query using Django ORM.

```python
# start_date = '2019-01-01'
# end_date = '2019-02-01'
User.objects.filter(
    member_since__gte=start_date,
    member_since__lte=end_date,
).annotate(
    date=TruncDate('member_since')
).values('date').annotate(count=Count('id'))
```

Quick explaining, this query filters all users with the field
`member_since` between the
`start_date` and the
`end_date`, after that creates a new
field called `date` with only the date
information, ignoring all hour and timezone information then extracts
all the dates to a queryset list and join to this date list the count of
the users of the day. The output of this query was:

```python
<QuerySet [
{'date': datetime.date(2019, 1, 24), 'count': 12},
{'date': datetime.date(2019, 1, 9), 'count': 26},
{'date': datetime.date(2019, 1, 14), 'count': 13},
{'date': datetime.date(2019, 1, 20), 'count': 81},
{'date': datetime.date(2019, 1, 1), 'count': 7}, <<---
{'date': datetime.date(2019, 1, 7), 'count': 59},
{'date': datetime.date(2019, 1, 17), 'count': 33},
{'date': datetime.date(2019, 1, 2), 'count': 17},
{'date': datetime.date(2019, 1, 5), 'count': 59},
{'date': datetime.date(2019, 1, 12), 'count': 65},
{'date': datetime.date(2019, 1, 19), 'count': 36},
{'date': datetime.date(2019, 1, 30), 'count': 81},
{'date': datetime.date(2019, 1, 11), 'count': 35},
{'date': datetime.date(2019, 1, 18), 'count': 48},
{'date': datetime.date(2019, 1, 25), 'count': 38},
{'date': datetime.date(2019, 1, 22), 'count': 37},
{'date': datetime.date(2019, 1, 13), 'count': 2},
{'date': datetime.date(2019, 1, 3), 'count': 34},
{'date': datetime.date(2019, 1, 29), 'count': 17},
{'date': datetime.date(2019, 1, 23), 'count': 29}, '...(remaining elements truncated)...']>
```

Everything seemed ok until I received a support request saying that the
numbers may not be correct.

On a dashboard, we have a graph with the number of *new members* *per
day* on the *last month*, plus a big card on the side of the graph with
the number of new members of the day, the bug report was that the graph
and the card number were not matching. To investigate if there was, in
fact, a bug I made a quick query to check the number of new users from
an individual day:

```python
# start_date = '2019-01-01'
User.objects.filter(
    member_since=start_date
).count()
# Result: 10
```

Numbers didn't match, the bug was real.

### Investigating the problem

As a developer I always fell that I'm partially a detective, catching
bugs, searching their hidden motives. My first thought was: let’s check these users membership dates more closely. With the same query of before with a small tweak on the date formatting the result is:

```python
# User.objects.filter(member_since=start_date).count() - Result: 10
['2019-01-01 01:16:36 +0000',
 '2019-01-01 01:22:41 +0000',
 '2019-01-01 01:52:39 +0000',
 '2019-01-01 10:32:11 +0000',
 '2019-01-01 13:31:50 +0000',
 '2019-01-01 14:04:57 +0000',
 '2019-01-01 17:07:00 +0000',
 '2019-01-01 18:24:38 +0000',
 '2019-01-01 19:42:22 +0000',
 '2019-01-01 20:54:14 +0000']
```

The results are the same as the previous individual query, showing a
total of 10 users per day. However, the interesting thing here is the
date format, more specifically the timezone part: `+0000`, this minor part indicates the timezone where the date
is saved. We can see that the dates are saved in UTC, we can also see
that there are 3 dates at the very beginning of the day.

Back to the old query, the filter by the field seems to be working
properly, the problem may be happening on the part of annotating a new
date without the hour and timezone information.

```python
# same query as above
User.objects.filter(
    member_since__gte=start_date,
    member_since__lte=end_date,
).annotate(
    date=TruncDate('member_since')
).values('date').annotate(count=Count('id'))
```

The `TruncDate` class extends from the
`Trunc` class which truncates a date up
to a significant component, in this case, the date component. Given the
datetime `2019–01–01 01:16:36 +00:00`,
the built-in `kind`s return:

-   day: 2019--01--01 00:00:00+00:00

But there's a catch if a different timezone like
`America/Recife` is active in Django,
then the datetime is converted to the new timezone before the value is
truncated, resulting in:

-   day: 2018--12--31 22:16:36 -03:00

```python
# Datetime conversion
local_tz = pytz.timezone("America/Recife")
local_dt = date.replace(tzinfo=pytz.utc).astimezone(local_tz)
new_date = local_tz.normalize(local_dt)
print(new_date.strftime('%Y-%m-%d %H:%M:%S %z'))
# Result: 2018-12-31 22:16:36 -0300
```

And we found the bug.

### Fixing the bug

Before the system would make two queries to get basically the same
information, **one solution** would be simply to use one query to
maintain consistency. **Other solution** would make all operations and
after that made the timezone conversion from UTC to the local timezone.
**The solution** I made was to annotate the date with the correct
timezone on the very beginning.

```python
# start_date = '2019-01-01'
# end_date = '2019-02-01'
User.objects.annotate(
    creation_date=TruncSecond('member_since')
).filter(
    creation_date__gte=start_date,
    creation_date__lte=end_date,
).annotate(
    date=TruncDate('creation_date')
).values('date').annotate(count=Count('id'))
```

Not the best solution, but was the less disruptive, at the moment, for
the system architecture.
