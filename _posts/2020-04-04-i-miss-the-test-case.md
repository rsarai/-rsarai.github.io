---
layout: post
title:  An overview of Python's TestCase class
categories: [python, testing]
excerpt: This week I missed the Django TestCase class. Yes, crazy right? Even though it's not a hot topic, full of mojo or buzz in the Django world, it should. The Django folks have made an incredible job with this feature/asset/code?/class and out of this mess I was very grateful to them. Django has done some amazing job in their TestCase, the base for this class is the unittest.TestCase of the language core implementation. Let's check this base class first.
---

Despite Brazilian stupid president false claims, I'm in quarantine. Stuck at home. Avoiding human contact. I'm thankful to be able to work remotely and thankful for the company I work for. It's amazing to work in a place that gives the workers support and confidence through this moment of crisis, at the same time that other "major" tech companies have been laying off employees. Hard times. Even harder for people who don't have the same support I do, autonomous workers in the country, or even people that have lost someone to this virus. I'm deeply moved by their situation and I wish things were better to all of them, wish we didn't have an irresponsible man as president, wish that the virus just took a vacation, and way more. This is a technical post though. It just seemed a bit wrong to start anything without mentioning all the situations we are living in. So much is different and I can't think the same way I used to.

&#x2013;

# Table of Contents
1.  [Python TestCase](#org5dcbf09)
    1.  [Setup work](#orge0c4b53)
    2.  [Helpers](#org4298d20)
    3.  [Assert Methods](#orgd143ae2)
    4.  [Extension Methods](#orga240f5c)
    5.  [Run, test Run](#orgf53fcbf)
2.  [Discussion](#org3021d1b)
    1.  [That's the place for my opinion](#org0f72d31)

Testing is important. It is interesting to see how the concern with this topic shifted to this generation of programmers. While reading books of interviews with amazing and senior programmers from the last century I noticed that when asked "Do you used to write tests?" the answers usually were somewhere around "Ain't nobody got the time for that". Of course that in modern times I still have conversations about when to write tests, what are the costs, and we have come up with a new punch line for not writing tests: "We will buy this tech debt now and will pay it later". However, I'm still haven't gone to a conference that didn't have a talk on testing or TDD. There's a growing market for tools that not only facilitate testing application code but the integration of your code with other services like browsers, OS, integrations. Even better, we are incentivized to write tests, is incredible how big frameworks like Django came with out of the box tools to improve testing. More and more I feel that we pass the point where tests were just a technical advantage, we are heading to a state where is a business requirement, one of the core stones to achieve stable and reliable products.

This week I missed the Django TestCase class. Yes, crazy right? Even though it's not a hot topic, full of mojo or buzz in the Django world, it should. The Django folks have made an incredible job with this feature/asset/code?/class and out of this mess I was very grateful to them. Django has done some amazing job in their ```TestCase```, the base for this class is the ```unittest.TestCase``` of the language core implementation. Let's check this base class first.


<a id="org5dcbf09"></a>

# Python TestCase

The TestCase class is a root class that only inherits from object and that's it. Head to <a href="https://github.com/python/cpython/blob/6467134307cf01802c9f1c0384d8acbebecbd400/Lib/unittest/case.py#L375" target="_blank">cpython repository</a> to view the source of code of this class. Just by reading the docstring, there is already useful information:

-   The instance of this class should define a single test case
-   The test code should be in a ```runTest``` method
-   You should subclass TestCase to write your tests
-   It is possible to create fixtures by overriding the 'setUp' and 'tearDown' methods
-   If it is necessary to override the ```__init__``` method, the base class ```__init__``` method must always be called

The need for all the methods to be in a ```runTest``` method is new to me. From the text, I supposed that one class can only have one test case method, although that's not something I do when writing my tests. Let's keep going and see if I can ensemble more information.


<a id="orge0c4b53"></a>

## Setup work

The ```__init__``` methods set the instance of the class plus some custom assertEqual functions by adding a type-specific constraint.

```python
self.addTypeEqualityFunc(dict, 'assertDictEqual')
# a few other bindings
# ...
```


<a id="org4298d20"></a>

## Helpers

There are private methods to control error messages, exception handling. They are:

- ```_addUnexpectedSuccess```
- ```_addExpectedFailure```
- ```_feedErrorsToResult```
- ```_formatMessage```


<a id="orgd143ae2"></a>

## Assert Methods

To test if something works we must compare the result from our code with the expected result. In the ```TestCase``` class there are implemented many assert methods.

-   assertNotEqual
-   assertEqual
-   assertAlmostEqual
-   assertSequenceEqual
-   assertIn
-   assertDictContainsSubset
-   assertCountEqual
-   assertLessEqual
-   assertIsNone
-   assertRaisesRegex
-   &#x2026;

And many others.


<a id="orga240f5c"></a>

## Extension Methods

We have talked about some of the implementations provided by the ```TestCase``` class. There are some methods without any implementation and that can be used to improve your test environment.

-   setUp
-   tearDown
-   setUpClass
-   tearDownClass

These methods can be used to construct and deconstruct fixtures.


<a id="orgf53fcbf"></a>

## Run, test Run

The ```run``` method orchestrates the class flow. The first thing done by this method is to prepare a <a href="https://github.com/python/cpython/blob/6467134307cf01802c9f1c0384d8acbebecbd400/Lib/unittest/result.py#L24" target="_blank">TestResult</a> object, this class acts as a holder for test results information. It is interesting to see that they modeled a different class to deal with the specifics of the result.

The test method is gathered from the ```self._testMethodName``` set on init which is "runTest" by default (<a href="https://github.com/python/cpython/blob/6467134307cf01802c9f1c0384d8acbebecbd400/Lib/unittest/case.py#L634" target="_blank">code</a>):
```python
testMethod = getattr(self, self._testMethodName)
```


After some setup work and some checks to assess if the class or method was skipped is time to run the test. The core logic is shown on the following snippet:

```python
        with outcome.testPartExecutor(self):
            self._callSetUp()
        if outcome.success:
            outcome.expecting_failure = expecting_failure
            with outcome.testPartExecutor(self, isTest=True):
                self._callTestMethod(testMethod)
            outcome.expecting_failure = False
            with outcome.testPartExecutor(self):
                self._callTearDown()

        self.doCleanups()
        for test, reason in outcome.skipped:
            self._addSkip(result, test, reason)
        self._feedErrorsToResult(result, outcome.errors)
```

Some important things on this snippet:

-   The ```outcome``` is an instance of the class <a href="https://github.com/python/cpython/blob/6467134307cf01802c9f1c0384d8acbebecbd400/Lib/unittest/case.py#L45" target="_blank">_Outcome</a>, another abstraction that exposes the ```testPartExecutor``` context manager. It was built to run with partial executions and deal with interruptions, skipped tests, expected failures.
-   ```_callSetUp```, ```_callTestMethod``` and ```_callTearDown``` are private methods used as interface to the volatile tests implementation.
-   The setUp and tearDown are widely used. However, there's still one more clean method that runs after the tearDown: ```doCleanups```
-   The ```doCleanups``` method works by executing the items in the ```_cleanups``` attribute, to fill this attribute one must call ```addCleanup``` passing a function and the necessary parameters. This cleanup step is useful in case you for some reason are using a broken setup. If the ```setUp()``` fails, the ```tearDown()``` will not be called, but still any cleanup functions added will be called.

Moving on with the code, the test is marked as expected failure or unexpected success. Finally the tests are stopped and the references are cleared. A simplified version of flow is:

-   Calls ```__init__``` passing the ```method_name``` the default is "runTest"
-   Calls setUp
-   Calls runTest
-   Calls tearDown
-   Calls cleanUp


<a id="org3021d1b"></a>

# Discussion


<a id="org0f72d31"></a>

## That's the place for my opinion

The TestCase provides an extensible framework to create complex and complete test suits. It provides you a well-defined flow to extend and a long list of assertion methods to validate business logic. Under the hood, it uses two main abstractions, one to deal with test results, appropriately called ```TestResult``` and one to deal with partial executions within the TestCase class named ```_Outcome```.

The ```TestResult``` class is almost a must for a programming language implementation I can see it's usefulness in a variety of scenarios. The most important for me is it separates the concerns, a separate class will deal with all the results of your tests, not need to couple this with the base test class. As a direct consequence, we ease future modification to the core developers to future users who may want, for example, to write a plugin to perform time-series analysis on test results data.

I also enjoy what the ```_Outcome``` class comes to the table. By checking the code we can see that the context manager ```with outcome.testPartExecutor(self):``` is used a few times through the class to control the flow and set a common ground between the tasks is the best part, elegant and pythonic.

&#x2013;

That's it. There's a lot more to testing in Python but we are not going to cover it all in this article. One loose end I left is the fact that the ```TestCase``` needs a ```method_name``` and if you did some testing with this module you probably never passed any ```method_name``` and you probably had lots on test cases in a single test class. That's something that the ```TestSuit``` class solves this for us, <a href="https://docs.python.org/3.0/library/unittest.html#organizing-test-code" target="_blank">here</a> you can find some further explanation.

