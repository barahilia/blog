---
layout: post
title:  "It's all bytes..."
date:   2014-10-16 15:03:00
categories: computers
---

Every developer knows that computers and programming languages operate on bytes.
Hard disks are measured in gigabytes, memory is addressed up to single byte,
double number takes 8 bytes and so on. Byte comprises 8 bits and can take value
from 0 to 255. But when it comes to some very spesific questions and actions we
halt and ask Google. And then copy ready recipes from
[StackOverflow](http://stackoverflow.com/) without really understanding all the
nuances. E.g.:

* How to read entire file to a string?
* How primitive data types are stored in memory?
* Why binary data from one computer looks like gibberish on another?

Let's try to describe some basic terms and to see how one may operate with them
in C++, Python (2.x) and C#.

