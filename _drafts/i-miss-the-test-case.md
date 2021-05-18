---
layout: post
title:  An overview of Python's TestCase class
categories: [python, testing]
excerpt: This week I missed the Django TestCase class. Yes, crazy right? Even though it's not a hot topic, full of mojo or buzz in the Django world, it should. The Django folks have made an incredible job with this feature/asset/code?/class and out of this mess I was very grateful to them. Django has done some amazing job in their TestCase, the base for this class is the unittest.TestCase of the language core implementation. Let's check this base class first.
---

1.  [Python TestCase](#org5dcbf09)
    1.  [Setup work](#orge0c4b53)
    2.  [Helpers](#org4298d20)
    3.  [Assert Methods](#orgd143ae2)
    4.  [Extension Methods](#orga240f5c)
    5.  [Run, test Run](#orgf53fcbf)
2.  [Discussion](#org3021d1b)

Testing is important. It is interesting to see how the perception around this topic shifted with this generation of programmers. I noticed, while reading an interview book with amazing senior programmers from the last century, that when asked "Did you use to write tests?" the answers usually were around "Ain't nobody got time for that". Forward to modern times. I still have conversations about when to write tests, what are the costs and the benefits, but everyone seems to agree on the importance of tests. I still haven't been in a conference that didn't have a talk on testing or TDD.

The impact of this cultural change is clear on the growing market for tools that not only facilitate testing code but also the integration of your code with other services like browsers, OSs, and APIs. Big frameworks like Django ship with out-of-the-box tools to facilitate testing. More and more I feel that we are past the point where tests were just a technical advantage, we got to the stage that it’s a business requirement, one of the cornerstones to build stable and reliable products.

This week I missed the Django TestCase class. I’m working on a <a href="https://en.wikipedia.org/wiki/Extract,_transform,_load" target="_blank">ETL</a> project , where we mostly do operations (many!) on databases. We need things to be fast, customizable and reliable, we also need to be certain that our features are working and continue to work while we make new features. Writing tests for this type of application is different from writing tests for a web application. For example, Django already comes with atomicity, parallel modes, concurrency safeguards, and all the structure to make tests with a connected database, it’s incredible how many issues are already solved by their setup. Writing tests without this type of support and for a different type of application was a challenge on it’s own with a lot of unexpected issues along the way. Today, I wanna talk about these built-in language features that can be useful in many situations, the Python base test class ```unittest.TestCase```. Let's check what is going on under the hood of the default test suite of the Python language.


<a id="org5dcbf09"></a>

# Python TestCase

Starting with an overview of the class. The TestCase class is a root class that inherits from ```object``` and that's it. Head to <a href="https://github.com/python/cpython/blob/master/Lib/unittest/case.py#L308" target="_blank">cpython repository</a> to view the source of code of this class. Just by reading the docstring, we can already find useful information:

-   The instance of this class should define a single test case
-   The test code should be in a ```runTest``` method
-   You should subclass TestCase to write your tests
-   It is possible to create fixtures by overriding the 'setUp' and 'tearDown' methods
-   If it is necessary to override the ```__init__``` method, the base class ```__init__``` method must always be called

The need for all the methods to be in a ```runTest``` method is new to me. From that text, I thought that a class could only have one test case method, although that's not something I do when writing my tests. Let's keep going and see if I can assemble more information.



<a id="orge0c4b53"></a>

## Setup work

The ```__init__``` method sets the instance of the class plus some custom assertEqual functions by adding a type-specific constraint.

```python
self.addTypeEqualityFunc(dict, 'assertDictEqual')
self.addTypeEqualityFunc(list, 'assertListEqual')
self.addTypeEqualityFunc(tuple, 'assertTupleEqual')
# a few other bindings
# ...
```


<a id="org4298d20"></a>

## Helpers

There are private methods to control error messages and exception handling. They are:

- ```_addUnexpectedSuccess```
- ```_addExpectedFailure```
- ```_feedErrorsToResult```
- ```_formatMessage```


<a id="orgd143ae2"></a>

## Assert Methods

To test if something works we must compare the result of our code with the expected result. In the ```TestCase``` class there are many options of assert methods.

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

We have talked about some of the default function implementations provided by the ```TestCase``` class. There are also function with no implementation that can be used to improve your test environment.

-   setUp
-   tearDown
-   setUpClass
-   tearDownClass

These methods can be used to construct and deconstruct fixtures.


<a id="orgf53fcbf"></a>

## Run, test Run

The ```run``` method orchestrates the class flow. The first thing done by this method is to prepare a <a href="https://github.com/python/cpython/blob/master/Lib/unittest/result.py#L24" target="_blank">TestResult</a> object, this class acts as a holder for test results information. It is interesting to see that they modeled a different class to deal with the specifics of the result.

The test method is defined at the ```self._testMethodName``` attribute, which is set  in the `__init__` method. By default it’s set to "runTest" (<a href="https://github.com/python/cpython/blob/master/Lib/unittest/case.py#L567" target="_blank">code</a>):

```python
def __init__(self, methodName='runTest'):
        """Create an instance of the class that will use the named test
           method when executed. Raises a ValueError if the instance does
           not have a method with the specified name.
        """
        self._testMethodName = methodName
```


```python
testMethod = getattr(self, self._testMethodName)
```

After some setup work and some checks to assess if the class/method was skipped is time to run the test. The core logic is shown in the following snippet:

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

-   The ```outcome``` is an instance of the class <a href="https://github.com/python/cpython/blob/master/Lib/unittest/case.py#L44" target="_blank">_Outcome</a>, another abstraction that exposes the ```testPartExecutor``` context manager. It was built to run with partial executions and deal with interruptions, skipped tests, expected failures.
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

The TestCase provides an extensible framework to create complex and complete test suites. It provides a well-defined flow to extend and a long list of assertion methods to validate business logic. Under the hood, it uses two main abstractions, one to deal with test results, appropriately called ```TestResult``` and one to deal with partial executions within the TestCase class named ```_Outcome```.

The ```TestResult``` class is almost a must for a programming language core implementation. I can see its usefulness in a variety of scenarios, the most important being the separation of concerns, a separated class will deal with all the results of your tests, not need to couple this with the base test class. As a direct consequence, we ease future modification to the core developers and to future users who may want, for example, to write a plugin to perform time-series analysis on test results data.

I also enjoy what the ```_Outcome``` class brings to the table. By checking the code we can see that the context manager ```with outcome.testPartExecutor(self):``` is used a few times through the class to control the flow and set a common ground between the tasks. And the best part, is elegant and Pythonic.

&#x2013;

That's it. There's a lot more to testing in Python but we are not going to cover it all in this article. One loose end left is the fact that the ```TestCase``` needs a ```method_name``` . If you did some testing with this module you probably never passed any ```method_name``` and you probably had lots of test cases in a single test class. That's something that the ```TestSuite``` class solves for us, the <a href="https://docs.python.org/3.9/library/unittest.html#organizing-test-code" target="_blank">documentation</a> is already a great place to start the investigation and for django users the explanation of what happens when you run manage.py test on a Django project also passes by the ```TestSuite``` class, you can find the explanation <a href="https://adamj.eu/tech/2020/09/05/what-happens-when-you-run-manage.py-test/" target="_blank">here</a>.
