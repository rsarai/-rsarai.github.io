---
layout: post
title: Solving Puzzles with Bitwise Operators
categories: [software]
excerpt: In this post I rant a bit about bitwise operators and review common ones.
last_modified_at:
---


In this post:
1.  [Rant on bitwise operations](#org37b4ede)
2.  [The problem](#orgf34dac3)
3.  [Solution #1](#orga41976e)
4.  [Solution #2](#org56cab37)
5.  [Links](#org0f310e4)

<br>

I don't really get bitwise operations. When to use them, and how. It's weird. I know they are essential for hardware people, I know that sometimes is the only way you have to achieve linear complexity, and still, I struggle.

Studying a bit about it this weekend, I spot a question: "Why is it so difficult to understand how and when to use bitwise operators in programming?" I can relate to that. To the first answer to this post, I can not: "It’s only difficult if you don’t understand how computers actually work". A peculiar and unhelpful claim, despite it, time to get back to the drawing board.


<a id="org37b4ede"></a>

## Bitwise operations

-   ``x << y``: Returns x with the bits shifted to the left by y places (and new bits on the right-hand side are zeros). This is the same as multiplying x by 2**y.


-   ``x >> y``: Returns x with the bits shifted to the right by y places. This is the same as //'ing x by 2**y.

-   ``x ^ y``: Does a "bitwise exclusive or". Each bit of the output is the same as the corresponding bit in x if that bit in y is 0, and it's the complement of the bit in x if that bit in y is 1.

-   ``x & y``: Does a "bitwise and". Each bit of the output is 1 if the corresponding bit of x AND of y is 1, otherwise it's 0.

-   ``x | y``: Does a "bitwise or". Each bit of the output is 0 if the corresponding bit of x AND of y is 0, otherwise it's 1.

-   ``~x``: Returns the complement of x - the number you get by switching each 1 for a 0 and each 0 for a 1. This is the same as -x - 1.
    -   Negative numbers always start with a 1. The smallest negative number is the largest binary value. 1111 is -1, 1110 is -2, 1101 is -3, etc
        -   Examples
            -   Find -4
                -   4 = 100
                -   Fill with zeros: 00100
                -   Invert: 11011
                -   Add 1: 11100

```python

    def is_odd(n):
        return n & 1 != 0

    def is_opposite(n, m):
        return (x ^ y) < 0

    def swap_vars(x: Int = 2, y: Int = 3):
        x = x ^ y
        y = x ^ y
        x = x ^ y

    def turn_off_kth(n, k):
        return n & ~(1 << (k - 1))

    def turn_on_kth(n, k):
        return n | (1 << (k - 1))

    def check_if_kth_is_set(n, k):
        """Returns 0 for not set, and the value for set"""
        return n & (1 << (k - 1))

    def toggle_the_kth_bit(n, k):
        """Works because of XOR
            n: 20 0b10100
            k: 2 0b10
            1 << (k - 1): 2 0b10
            n ^ (1 << (k - 1)): 22 0b10110
        """
        return n ^ (1 << (k - 1))

    def unset_rightmost_set_bit(n):
        # n: 49 0b110001
        # n - 1: 48 0b110000
        # n & (n - 1): 48 0b110000
        return n & (n - 1)

    def is_power_of_2(n):
        # returns 0 if is power of 2
        return n & (n - 1)

    def find_index_of_rightmost_set_bit(n):
        if (n & 1):
            # is odd number (ends with 1)
            return 1

        x = (n & (n - 1)) ^ n

        pos = 0
        while n:
            n = n >> 1
            pos = pos + 1
        return pos

    def find_position_of_only_set_bit(n):
        x = n & (n - 1)
        if x != 0:
            return "Invalid"

        pos = 0
        while n:
            n = n >> 1
            pos = pos + 1
        return pos

    def get_number_of_1(n):
        parity = 0
        while n:
            n = n & (n - 1)
            parity = parity + 1
        return parity
```

<a id="orgf34dac3"></a>

## The problem

All this rant started with the following problem: Given an array of integers where every integer occurs three times except for one integer, which only occurs once, find and return the non-duplicated integer.

-   Input - [6, 1, 3, 3, 3, 6, 6]
    -   Output - 1

-   Input - [13, 19, 13, 13]
    -   Output - 19

Do this in O(N) time and O(1) space.

Obs: The problem was extracted from [Daily Interview Problem / Former Pinterest COO accuses the company of Gender Bias!](https://blog.quastor.org/p/daily-interview-problem-former-pinterest-coo-accuses-company-gender-bias)


<a id="orga41976e"></a>

## Solution #1
```python
    def find(l):
        for i in l:
            c = 0
            for comparing_i in l:
                if i == comparing_i:
                    c = c + 1

                if c > 1:
                    c = 0
                    break

            if c == 1:
                return i
        return -1
```
I have tests and all of them are passing. However, this is not the optimal solution.


<a id="org56cab37"></a>

## Solution #2

I still find the following solution a bit weird. I really don't get bitwise operations.
```python
    def find(nums):
        result_arr = [0] * 32
        for num in nums:
            for i in range(32):
                bit = num >> i & 1
                result_arr[i] = (result_arr[i] + bit) % 3

        result = 0
        for i, bit in enumerate(result_arr):
            if bit:
                result += 2 ** i

        return result
```
--------------------
After some thought and study of the operations highlighted before, the logic becomes clear. Still, I don't think this solution would come to me in an interview, I would stay with my old Python and write a solution fully tested in 10 minutes.


<a id="org0f310e4"></a>

## Links

-   <https://www.techiedelight.com/bit-hacks-part-1-basic/>
-   [Bit Manipulation: Interview Questions and Practice Problems](https://medium.com/techie-delight/bit-manipulation-interview-questions-and-practice-problems-27c0e71412e7)
-   <https://graphics.stanford.edu/~seander/bithacks.html>


