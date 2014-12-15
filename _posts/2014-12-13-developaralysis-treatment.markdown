---
layout: post
title:  "Developaralysis treatment"
date:   2014-12-13 21:40:00
categories: computers
---

Today I've read an article
[You Too May Be A Victim Of Developaralysis](http://techcrunch.com/2014/10/18/you-too-may-be-a-victim-of-developaralysis/).
The text caused me to think about some usual and established things in a new
light. With the evidence of beyond ten thousands likes and tweets, many others
might have shared my impression. After some consideration I decided that no,
I'm not a victim. I feel good with a wealth of existing languages, libraries,
frameworks and systems. I do pursue new ideas and techniques but I'm not harmed
by incompetence in all of them or even the best of them.

Probably, if you are a victim, there's a treatment for developaralysis. As with
most modern undertakings, let's start with Wikipedia:

> People with the flu are advised to get plenty of rest, drink plenty of
> liquids, avoid using alcohol and tobacco and, if necessary, take medications
> such as acetaminophen (paracetamol) to relieve the fever and muscle aches
> associated with the flu.

-- from <http://en.wikipedia.org/wiki/Influenza#Treatment>.

Humor aside. With profession, nothing comes as easy as cure to flu. And if all
you want is to take a rest and paracetamol, then better forget about software.
The real treatment is really simple to write, but very difficult to apply. It
is:

* learn in depth than in breadth
* respect good enough solutions

**TODO**:

* read a computer language is really simple; learn a new - not much harder
* this is because of a limited number of concepts, principles that are same
* optimizations, ideas come once and again under different names and wrappers
* but if you understand them deeply, you'll see the same core
* Microsoft released a new true platform once 2 years: COM, COM+, ActiveX, .NET,
WinForms, WPF, WinRT - but should you keep up with all of them?
* our current project is in Web - we learn environment on the way at a good
progress
* npm, bower - all other package managers, are not new under the sun; Unix had
rmp for dozens of years, nuget, etc - are essentially the same: they bring order
into packages and track their dependencies
* no need to learn everything; once you learned in depth you will understand the
good and the bad fast enough and you may stop at the first appropriate good
station

Take a developer experienced with Java that
by bad chance must start working in C\#. Oh crap! He was used to write
`System.out.println("hi")` but now he learns `Console.WriteLine("hi")`. And the
old good looping `for (String s : arr)` becomes unclear `foreach (var s in arr)`.
And he even didn't really start to harness standard libraries. And yes, good
programmers write tests first so we should go and learn `nUnit` after nice and
simple `jUnit` that he is accustomed to. Dummy `Assert.AreEqual(1+2, 3)` instead
of normal and clear `assertEquals("1+2=3", 1+2, 3)`.

By now you might be laughing to yourself, or calling this rubbish. So I'll
continue to my point. Yes, I brought those examples to show, that essentially,
both C\# and Java are not so intrinsically different. They are two languages
both in their own rights, with mostly not intersecting communities. But still
for one coming from Java it'll not take the same to be effective in C\# as for
those coming from no background at all. Or will it? Alice - sorry for bringing
security names here - is an good learner. When she first met unit tests in her
life, she was astonished, and started learning and experimenting what this is
about. And she quickly understood that there are a couple of well known players
in that field: suites, tests, asserts. Probably later she got to "system under
test", starting up and tearing down procedures. Bob is quite different. He's
renowned sprinter, accomplishing first most tasks. He quickly and efficiently
searches the Internet, copies and adapts relevant code parts, uses libraries
and finds well-hidden functions that boost his productivity. He don't stop to
contemplate over how things are done, what is the theory. He looks at examples
and quickly learns that in most of them there are suites, tests and asserts and
when the need arises, he comes to start up and tear down things.




