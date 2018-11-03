---
layout: post
title:  "Relativistic unit test dilation"
date:   2018-11-12 14:50:00
categories: computers
---

Sorry to disappoint you, if you expected references to Einstein and Lorentz. The
story is about testing in Python. Actually the subtitle is:

## Subtitle: unit test runs longer that it runs

New feature or some quick experiment was on its way, I finished as small as
possible touch on code in PyCharm and ran the tests to ensure nothing is broken.
Of course it was. As usual in such event, I prefer to debug the changed code
behavior at some specific test. I picked one, and just ran it again alone, to
"warm up" the command line. And then it happened:

```
$ ./test_suite.py MyTest.test_me
F
======================================================================
FAIL: test_me (__main__.MyTest)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "./test_suite.py", line 7, in test_me
    self.assertTrue(False)
AssertionError: False is not true

----------------------------------------------------------------------
Ran 1 test in 0.001s

FAILED (failures=1)
```

Er. Stop looking at the output above. It is completely OK, nothing juicy. I did
not tell you I saw the problem, but rather I *experienced* it. And then I asked
Bash to verify my feelings. The output was:

```
$ time ./test_suite.py MyTest.test_me
F
# truncated
----------------------------------------------------------------------
Ran 1 test in 0.001s

FAILED (failures=1)

real    0m1.145s
user    0m x.xxxs # irrelevant
sys     0m x.xxxs
```

Now you can look. And see. The test ran a *millisecond* or so, but the entire
command took more than a *second*. Why the test runs longer... than it runs?
Apparently there is some spot either in tests or in the project that does
something, hopefully useful, during that *second*. This is possibly an
appropriate moment to confess in you. Tests in that project, they are not really
fast. I mean the unit tests, and one waits for about half a minute for the
entire suite to complete. This sucks, but again, not enough. And I was telling
myself for a long period of time to reduce timing, and it had never been
important and impelling enough. I'm telling this now to explain, why that
*second* wasn't noticed earlier, immediately when it was introduced.

How to find out, what a process does in that *second*? Profiler to help. The
suite runs in a pretty simple manner with a built-in Python framework. Of
course, you can be used to `Nose`, or advocate for `pytest`, which I prefer
using myself recently. But anyhow, the built-in `unittest` is good enough.
Usually, it goes like the following:

```python
from unittest import TestCase, main

# tests implementation comes here

if __name__ == '__main__':
    main()
```

We have an explicit entry point to profile:

```python
if __name__ == '__main__':
    from cProfile import run
    run('main()')
```

All nice and fun, just the answer is not found:

```
----------------------------------------------------------------------
Ran 1 test in 0.014s

FAILED (failures=1)
         265 function calls (262 primitive calls) in 0.035 seconds

   Ordered by: standard name

   ncalls  tottime  percall  cumtime  percall filename:lineno(function)
        1    0.000    0.000    0.035    0.035 <string>:1(<module>)
        1    0.000    0.000    0.000    0.000 case.py:178(__init__)
...
```

Meaning, the profiler was unable to catch the offending method, that apparently
is invoked somewhere at start up or at importing. Two approaches can be used to
overcome this. First the profiler can run itself the entire script, like this:

```
$ python -m cProfile ./test_suite.py

----------------------------------------------------------------------
Ran 0 tests in 0.000s

OK
         861 function calls (849 primitive calls) in 1.028 seconds

```

Here the added *one second* is detected properly, meaning the start up or
importing logic is invoked and is measured by the profiler. But for some reason
the tests do not run. Apparently the initialization logic for profiling and for
testing clash in some way. For this reason and for a couple of others, I
personally prefer to explicitly run the profiler directly from a code.

How to do it here? Before actual execution of the tests one by one and rendering
a nice report, the tests framework should get a list of those tests. This stage
is called "discovery". In `unittest` it is usually done implicitly, but can be
called directly by a user as following:

```py
loader = TestLoader()
# start the search at the current working directory
return loader.discover('.')
```

The snippet above can be wrapped as a function and profiled. And, voila, this
does help:

```
$ ./test_suite.py
         1080 function calls (1077 primitive calls) in 1.036 seconds

   Ordered by: standard name

   ncalls  tottime  percall  cumtime  percall filename:lineno(function)
        1    0.000    0.000    1.036    1.036 <string>:1(<module>)
...
        1    0.000    0.000    1.036    1.036 loader.py:145(discover)
...
        1    0.000    0.000    1.001    1.001 test_suite.py:12(make_data)
        1    0.000    0.000    1.001    1.001 test_suite.py:21(OtherTest)
```

And the culprit gets detected. It is `make_data()`, which was called from the
test case `OtherTest`. Let us look at them more closely:

```py
def make_data():
    # please bear with me; yes, this is contrived, and comes
    # instead of somewhat heavy computation in the real project
    sleep(1)


class OtherTest(TestCase):
    data = make_data()

    def test_other(self):
        self.assertTrue(self.data)
```

Finally, the "aha!" moment. The problem is with the class member `data` that is
being initialized by some long running function. This happens at import time or
at discovery stage. And it will run and take its time, even if the `OtherTest`
is not used at all. Not good. Fortunately, `unittest` supports `TestCase`
initialization for stuff common to all tests:

```py
class OtherTest(TestCase):
    @classmethod
    def setUpClass(cls):
        cls.data = make_data()

    def test_other(self):
        self.assertTrue(self.data)
```

Yay. Timing is good again:

```
$ time ./test_suite.py MyTest.test_me
Ran 1 test in 0.001s

FAILED (failures=1)

real    0m0.095s
user    0m0.064s
sys     0m0.024s
```

Basically, this is the end. The happy end to the story. But I can't refrain but
to grumble a bit at frameworks. A good unittest harness allows to get the fruits
early and at low cost. Same thing with UI or Web server. Just when misbehavior
sneaks in, it might be difficult to attach the diagnostic tools to the spots in
or under the frame.

As for me, I will continue running tests in a framework. And now I've learned
and shared with you another bit of Python workings in class initialization.
