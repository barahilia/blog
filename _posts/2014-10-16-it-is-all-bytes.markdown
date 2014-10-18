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

This post intends to shed the light on how data represented in bytes looks like
in different mediums. But how to make bytes of data? For this encoding is used,
and there's at least one for every type of data. Here this is assumed to be
known or some relevant details are explained on the way.

## Notation

We'll make use of
[hexadecimal C-style notation](http://en.wikipedia.org/wiki/Hexadecimal#Using_0.E2.80.939_and_A.E2.80.93F)
for a byte value: `\x00`, `\x01`, ..., `\xff`. So a sequence of 3 bytes with
decimal values 10, 16, 254 will be written as `\x0a\x10\xfe`.

## Basic terms

An unsigned "short" 16-bit integer requires 2 bytes. E.g. since
`1024 = 4 * 256 + 0` the number 1024 can be represented with 2 bytes with
values: 4 and 0. But how exactly do we write them? Is it `\x04\x00` or
`\x00\x04`? It occurs that both is possible and this is what
[*Endianness*](http://en.wikipedia.org/wiki/Endianness) is all about. The first
variant `\x04\x00` is called *big-endian*; it is used in Motorola 68000 and
PowerPC and some network protocols. The second variant '\x00\x04' is called
*little-endian* and is used in Intel x86 and AMD64.


