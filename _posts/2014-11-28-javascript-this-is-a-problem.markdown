---
layout: post
title:  "JavaScript: 'this' is a problem"
date:   2014-11-28 16:40:00
categories: computers
---

**Disclaimer**: As of now, I'm a newbie to JavaScript with bare year of
industrial experience in the language and its environments. I do base tips and
conclusions on a dozen years with other languages. So take my advice with a
grain of salt.

For keen-eyes who decide in a split second if to pursue with a post, I shall
make my advice really short: don't write *this* in your code unless you need to.
And if it sounds not trivial, there will be some concrete samples.

Both in real code and in well-established libraries one can find something
resembling:

    function MyObj() {
        this.a = 0;
        this.count = function () {
            this.a += 1;
        };
    };

No need to continue, it's known that `count()` function works at the scope of
`MyObj` only and it can't be copied or send as a parameter. If you do such a
thing and encounter a bug, a simple well-known technique comes handy and is used
all the time: `var that = this;`. Meaning, capture the object pointed by *this*
in `that` variable, which in turn is captured by `count()` function.

In JavaScript there are (at least) 4 methods to invoke a function that differ
by how *this* is set.

**TODO**: insert a list, reference to John Resig

Just to be aware of them when you write your code is intimidating. And if you
look around, a substantial volume of text in books and articles explain, how
exactly all such invocation methods work and how to get the code right. Do
yourself a favor and read about *this*. The theme appears in many advanced
libraries and the concept is crucial for understanding and mastering JavaScript.
You will need this knowledge to understand some patterns and find solutions in
life situations. But such a complexity should raise a red flag: not for
everything. Advanced tools should wait patiently on a shelf.




