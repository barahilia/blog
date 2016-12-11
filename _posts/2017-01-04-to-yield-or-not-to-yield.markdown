---
layout: post
title:  "To yield or not to yield"
date:   2017-01-04 19:00:00
categories: computers
---

## When and how

Single object is a simple thing to work with. Not so easy with collections.
Abundance come at a cost. There are arrays, and lists, and dictionaries or maps,
and sets and hashes... To access them all iterators were conceived and special
kinds of loops were invented which are basically a syntactic sugar for
iterators. The only thing you need to work with collections is to get their
items one by one and to know when items are exhausted. And this precisely
defines minimal interface for the iterator and also a requirement: to remember
current position inside the underlying collection. Classic object-oriented
languages like C++ or Java choose implementing iterators as an object,
encapsulating both the position and the reference to the collection.
While this allows for clean and convenient usage, it's quite tedious to write
another iterator class in addition to every collection. And also the logic
of the `next()` function gets split to pieces by advancing only one step at a
time.

It is wonderful that another approach was suggested a long time ago in
[CLU](https://en.wikipedia.org/wiki/CLU_(programming_language))
programming language that first appeared in 1975 more than 40 years ago, well
before C++ and Java. It was designed by
[Barbara Liskov](https://en.wikipedia.org/wiki/Barbara_Liskov) and her students,
the very same person behind the Liskov substitution principle, which stands for
_L_ in [SOLID](https://en.wikipedia.org/wiki/SOLID_(object-oriented_design))
principles. One key feature of CLU are iterators that can provide values via
_yield_ statement. Today in Python we can write similarly:

```python
def iterate(l):
    for value in l:
        yield value

def main():
    for a in iterate([1, 2, 3]):
        print a
```

Yes, yes, this only illustrates the idea, because of course there's a default
list iterator in Python. Below there are examples showing why to use or why not
to use yield in code. The discussion is mainly relevant to Python, though the
ideas may apply to other languages too.

## Clean, to the point

Compare the following two functions:

```python
def double_a(collection):
    output = []

    for item in collection:
        output.append(item * 2)

    return output

def double_b(collection):
    for item in collection:
        yield item * 2
```

In most cases caller can invoke any of the two variants above oblivious to the
fact whether it gets back a list or a generator. But the functions are a little
different. `double_b()` is very direct about its actions: it iterates over given
collection and doubles the items. `double_a()` does the same, but it also has
one more task: to save all the results in a list that will be returned at the
end.

What the caller does with the result? It may print it to console, or it may
calculate the sum and the average of the doubled collection, or it may convert
it to a set. When we are writing the doubler functionality, better not to
consider how the result will be used and not to provide any specific data
structure. The responsibility doesn't lay here. And generator achieves that.
Also the second function is more readable. There's one variable less to keep
attention to, one variable less to name.

Now look at Fibonacci numbers computation:

```python
def fib():
    a = b = 1

    while True:
        yield a
        a, b = b, a + b
```

This function - rather unusually - doesn't accept the `n` parameter, specifying
which Fibonacci number or how many of them to produce. Instead, it is
responsible to compute the numbers and to implement the algorithm. It is up to
the user to decide if she needs a sequence or a single number or anything else
and how many of them.

## Unbound sequences

Usually in programming we deal with concrete sizes. Generators allow us to
produce sequences of unbound size - `fib()` is a nice example. More of them can
be found in the standard Python
[itertools](https://docs.python.org/2/library/itertools.html) module. I prefer
_unbound_ to _infinite_: computers are physical devices and there's nothing
infinite about them, while unbound assumes finiteness. It means the language
construct doesn't put any constraint on the size. Given enough time we can get
to any point in the sequence.

## One way ticket

By now you're sure any function returning a list can be made generator. Let's
break this belief:

```python
def a():
    return [0, 1]

def b():
    yield 0
    yield 1

la, lb = a(), b()

print la, list(lb)  # [0, 1] [0, 1]
print la, list(lb)  # [0, 1] []
```

See that, ah? Just like iterators, generators produce items one by one, until
there are no more and then it stops and yields no more. While `a()` returns
items in the list that can be printed or copied or used in any other way time
and again, the return value of `b()` can be used only once.

Usually this isn't an issue. But if it is, we may consider the following trick:

```python
def _b():
    yield 0
    yield 1

def b():
    return list(_b())
```

Generally speaking, if the function should return a list and it can be done with
list-comprehension or `map()` or in some other direct way - all right. But once
`list.append()` is involved, yield statement can achieve cleaner code and the
trick above can still ensure a normal list is returned. And if such _listify_
operation becomes frequent, a decorator can be developed. See examples
[here](http://stackoverflow.com/questions/12377013/is-there-a-library-function-in-python-to-turn-a-generator-function-into-a-functi)
and
[here](https://chezsoi.org/lucas/blog/2016/02/11/en-useful-short-python-decorator-to-convert-generators-into-lists/).

## Fail early

Another non-obvious interface benefit of generators is a possibility to split
between the computation and the validation. Assuming we have a large set of data
to compute, which is then passed to another function or saved to a file. If all
the items are OK - well done. If not, we should fail early e.g. by throwing an
exception.

If the computation result is saved in a list, verification must be done in
parallel to computation in the same function. Which means additional
responsibility, longer code and even it is not always possible - what if the
computation happen in some external library, that is completely unaware of
verification. And again yield solves the problem:

```python
def long_computation():
    while condition:
        # ...
        yield next_item

def compute_and_verify():
    for item in long_computation():
        if not is_good(item):
            raise ValueError(item)

def use_computation():
    for item in compute_and_verify():
        # use it somehow
        pass
```

## Performance

Finally we came to this. To me it is more important that generators make code
more elegant. Performance is usually relevant to smaller hot spots.

The idea is simple: in comparison to list interface where computation results
are first saved and are returned only at the end, "yielding" could reach faster
code and reduce memory usage. As a quick simulation we may use `%timeit` magic
command in [IPython](https://ipython.org/) with the following two functions:

```python
def f1():
    i = 0
    while i < 100:
        yield i
        i += 1

def f2():
    i, l = 0, []
    while i < 100:
        l.append(i)
        i += 1
    return l

In [3]: %timeit sum(f1())  # 10000 loops, best of 3: 59.6 µs per loop

In [4]: %timeit sum(f2())  # 10000 loops, best of 3: 99.2 µs per loop
```

So even for a rather small list of 100 items only, generators run almost twice
faster. Demonstration of memory usage would be a trickier thing, so I will leave
this out.

## Debugging quirks

Well, now that we looked into a number of examples for advantages, we should
investigate the disadvantages too. It is fair and it justifies the second part
of the post name - "... or not to yield". In my experience, debugging was the
most inconvenient thing about generators. Look at this:

```python
def f():
    a, b = 5, 8
    import pdb; pdb.set_trace()
    yield a
    a, b = b, a + b
    yield a

for item in f():
    print item
```

We need to debug `f()` and the problem hides somewhere in between. So we step
with the debugger from line `yield a` and instead of getting to the next line of
`a, b` assignment we exit to the calling loop. Only 4 steps later we will reach
the required statement in `f()`. In this tiny example this is a small issue, but
in larger project debugging gets more complicated.

Two techniques comes to aid. The first is: set a breakpoint in the line after
yield before leaving the function. Important note: sometimes more than one
iteration over `f()` runs simultaneously. And such scenario doesn't have to
involve multithreading. Just imaging how `zip(f(), f())` is executed.

Another technique is to replace the function call with `list(f())`, directly or
with a decorator as explained above. Debugger will still exit the function at
yield statement but will immediately get back to the flow.

## Profiling interpretation

Consider the following example:

```python
def worker():
    yield sum(range(1000000))

def middle():
    return worker()

def user():
    return sum(middle())

print user()
```

Basically, we have `user()` calling `middle()` calling `worker()`. Let's profile
it with `python -m cProfile main.py`:

```
   ncalls  tottime  percall  cumtime  percall filename:lineno(function)
        1    0.000    0.000    0.556    0.556 main.py:1(<module>)
        2    0.036    0.018    0.555    0.278 main.py:1(worker)
        1    0.000    0.000    0.000    0.000 main.py:4(middle)
        1    0.000    0.000    0.556    0.556 main.py:7(user)
```

There are at least two things that are correct, logical and sound, but rather
non-intuitive. The `worker()` function is called only once, but its ncalls is 2.
First it is called from `middle()` and a generator object is returned, no callee
code executed yet. After that, first iteration runs `worker()` code till the end
of the single yield statement. Next code after the yield runs. But there is no
more code, so `worker()` will return and the generator will emit
`StopIteration`. Not so obvious, isn't it.

Another bizarre thing is the cumulative time of `middle()`. The function
executes another function with `return worker()` and it could be expected that
its cumulative time to be greater or equal than that of the `worker()`.
But it is 0 seconds only. Again, because the actual call to `worker()` only
creates a generator object which is immediately returned to `user()`, where it
is really executed.

## Zero sequence generator

Well, I have to admit, this a purely syntactic complaint: there is no simple
obvious way to write a generator that returns an empty sequence. The need is
quite frequent when you create sequences this way and your interfaces expect
them, e.g. in case of:

```python
def func(data):
    for a in data():
        process(a)
```

Here `data` is expected to be a generator and so is called with parentheses
syntax and the return value is iterable. The catch is that empty function does
not work:`

```python
def a():  # Returns None
    pass

func(a())  # TypeError: 'NoneType' object is not iterable
```

Read
[Python Empty Generator Function](http://stackoverflow.com/questions/13243766/python-empty-generator-function)
on Stack Overflow for a detailed discussion and suggestions. In my opinion they
all look rather contrived.

An opposite of yielding nothing is to yield more than one thing - indeed quite a
common task. I would be happy to complain about that too, if not for
`yield from`, which was introduced in
[Python 3.3](https://docs.python.org/3/whatsnew/3.3.html#pep-380).
The fact that I still continue with Python 2 is my problem, isn't it?

## To summarize

We have seen a number of points for and against using the yield statement. Here
is a quick recap.

Why to use:
* reduce responsibility for data accumulation
* define unbound sequences
* get the ability to fail early
* reach better performance

Why not to use:
* one way ticket; reusing generator gets empty sequence
* more difficult debugging
* no simple syntax to yield nothing (and no yield many in Python 2)
