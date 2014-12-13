---
layout: post
title:  "JavaScript: 'this' is a problem"
date:   2014-11-28 16:40:00
categories: computers
---

**Disclaimer**: As of now, I'm a newbie to JavaScript with barely a year of
industrial experience in the language and its environments. But I do base my
ideas and conclusions on a dozen years of practice with other languages. Still
take my advice with a grain of salt.

For keen-eyes who decide in a split second if to pursue with a post, I shall
summarize the idea:
**don't write *this* in your code unless you need to.**
And if it sounds not trivial, the remaining part presents some concrete samples
and explanations.

Both in real code and in well-established libraries one can find something
resembling:

    function MyObj() {
        this.a = 0;
        this.count = function () {
            this.a += 1;
        };
    };

Let's try to use it:

    var wrong = MyObj(); // forgot new... happens
    var o = new MyObj(); // all right
    o.count();           // works properly
    var b = o.count;     // frequent, e.g. to register as a callback
    b();                 // does not work: this.a is undefined

It's well known and trivial to some, that `count()` function works at the scope
of `MyObj` only and it can't be copied or sent as a parameter. If you do such a
thing and encounter a bug, a simple established technique comes handy:
`var that = this;`. Meaning, capture the object pointed by *this* in `that`
variable, which in turn is captured by `count()` function.

In JavaScript there are many methods to invoke a function. John Resig in the
["Secrets of the JavaScript Ninja"](http://www.amazon.com/Secrets-JavaScript-Ninja-John-Resig/dp/193398869X)
makes a list of four (!) of them by how *this* is set:

+ as a function, directly on current scope, *this* is the same as at environment
+ as a method, on an object, *this* equals to the object state
+ as a constructor, when new object is generated and *this* set to it
+ via `apply()` or `call()` methods, where *this* can be set to anything

Just to be aware of all variants when you write your code is intimidating. And
if you look around, a substantial volume of text in books and articles explain,
how exactly all such invocation methods work and how to get the code right. Do
yourself a favor and read about *this*. The theme appears in many advanced
libraries and the concept is crucial for understanding and mastering JavaScript.
You will need this knowledge to understand some patterns and find solutions in
life situations. But such a complexity should raise a red flag: not for
everyday. Advanced tools should wait patiently on a shelf for a challenge.

So why the usage of *this* in other languages, Java for example, is valid?
Actually, the very resemblance of *this* in JavaScript to similar concepts in
other languages is deceptive. In Java, each object originates from a class and
has a state. All object's members including functions are defined per class and
when execute rely on the state of the object on which they operate. In
JavaScript on the contrary, there are no classes and functions are objects of
their own. Functions bear no link to the object in which they are defined, but
can operate on any state. This is not to make your life harder, but rather to
allow for a very nice and flexible language construct. So Java's *this* is
simple and well-defined pointer to object's state. But in JavaScript *this* only
seems to point to the scope in which its expression was written, while at
runtime can point to anything.

If you need just to create an object, prefer the simplest syntax:

    var obj = {};
    obj.a = 0;
    obj.count = function () { obj.a++; }

Some textbooks and libraries define all object members inside curly brackets:

    var obj = {
        a: 0,
        count: function () {
            obj.a++;
        }
    };

I use such a syntax for the shortest definitions only. On the one hand, defining
`count()` inside `obj` adds one level of indentation. On the other, for larger
objects the opening line might be left far above and it would
be easier for code observer to grasp `obj.count` than `count: ...` with the
hidden `obj =`. Both aspects harm readability so I opt to the former construct.

You need a constructor - wrap object definition in a function:

    var createObj = function () {
        var obj = {};
        var a = 0; // private object member
        obj.count = function () { a++; }
        
        return obj;
    };

Note that overhead is just two more statements: one to declare the object and
another to return it. Now, what do we gain:

* Very simple syntax, that even novice may write and use correctly
* Guaranteed same result under any invocation type
* No need to capture the state of object (actually it's done with `var obj`)
* No chance for missing `new` like with the constructor syntax

Here I introduced a private member through captured local variable. It is not
counted to advantages of this method, since this trick can be used inside
constructors. But probably there it "feels" less correct: members should be
defined through *this*, shouldn't they?


