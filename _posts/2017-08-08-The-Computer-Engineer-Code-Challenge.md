---
layout: post
title: #1 The Computer Engineer Code Challenge
categories: [code]
excerpt: ""
---

_Originally published on medium_

![](https://cdn-images-1.medium.com/max/800/1*dhvBPeb6bYqQU_KqlaboDQ.jpeg)

To begin this challenge I would love to write algorithms about [Hopfield
Net](https://medium.com/100-days-of-algorithms/day-80-hopfield-net-5f18d3dbf6e6) or the [Hanoi
Tower](https://medium.com/100-days-of-algorithms/day-61-hanoi-tower-ii-43bf3d19fe6a) and I will write, just will be a little bit later.
Right now, I'm a backend developer striving to become a full stack
developer, working with Django, React JS and other technologies of the
moment. I'm in a new job and haven't yet mastered the
Python's/Javascript's world. For summarize I need to work on my
programmer skills.

With that goal in mind, I chose spent the last month exercising my
programming skills in [CodeFights
website.](https://codefights.com/signup/asE3bgg9mnXW6KTvS/main) CodeFights is an excellent tool for training and
offers different types of practice like:

-   ***Interview Practice:*** Makes it easy to prepare for technical
    interviews
-   ***Company Bot:*** Your chance to challenge programming bots
    trained by engineers from top tech companies]
-   ***Arcade:*** Is a series of tasks to unlock new levels and
    locations.
-   ***Head-to-Head:*** Pretty much a battle against your friends and
    strangers
-   ***Tournaments:*** Real-time coding competitions (Best of
    all).

At first, I was playing the arcade mode, confess, I felt really smart,
but this feeling decrease when the challenges became more difficult, let
me show you:

1.  Given a year, return the century it is in. The first century spans
    from the year 1 up to and including the year 100, the
    econd --- from the year 101 up to and including the year 200,
    etc.




```python
# my solution:
def centuryFromYear(year):
    if year % 100 == 0:
        return year/100
    else:
        return int(year/100) + 1
```

Does it look easy right? But let up the level a little bit

2\. Given a decimal integer `n`, find an
integer `k ≥ 2` such that the
representation of `n` in base
`k` has the maximum possible number of
zeros. If there are several answers, output the smallest one.

-   For `n = 9`, the output should be\
     `maxZeros(n) = 2`.\
     `9 = 10012 = 1003 = 214...`\
     If you\'ll try all other bases, you\'ll see that the maximum
    possible number of zeros in these representations is
    `2`, thus the answer is
    `k = 2`.


```python
# best solution
def maxZeros(n):
    maxZ = -1
    maxB = -1
    for base in range(2, n+1):
        num = n
        tot = 0
        while num > 0:
            if num % base == 0:
                tot += 1
            num //= base
        if tot > maxZ:
            maxZ = tot
            maxB = base

    return maxB
```

If you think these challenges were easy, imagine having 10 minutes to
solve 3 challenges with this same degree of difficulty and don't forget
the competition of 4 or 5 other developers.

At first, you won't finish the 3 challenges, you progress gradually and
when you see it, the goal is not just to solve the issues, but to solve
in the best way possible. And that's when you research and check how
your opponents crack the challenge and go to the documentation, forums
and etc.

That's all folks and join me on
[CodeFights](https://codefights.com/signup/asE3bgg9mnXW6KTvS/main)!
