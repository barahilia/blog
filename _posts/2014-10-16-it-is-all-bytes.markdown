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

Let's try to describe some basic terms and to see how one may program them in
C++, Python (2.x) and C#.

This post intends to shed the light on how data represented in bytes looks like
in different mediums. But how to make bytes of data? For this *encoding* is
used, and there's at least one for every type of data. Here this is assumed to
be known or some relevant details are explained on the way. As well to keep
the post shorter, some more advanced aspects of discussed concepts are
deliberately ignored.

## Notation

We'll make use of
[hexadecimal C-style notation](http://en.wikipedia.org/wiki/Hexadecimal#Using_0.E2.80.939_and_A.E2.80.93F)
for a byte value: `\x00`, `\x01`, ..., `\xff`. So a sequence of 3 bytes with
decimal values 10, 16, 254 will be written as `\x0a\x10\xfe`.

## Basic terms

### Endianness

An unsigned "short" 16-bit integer requires 2 bytes. E.g. since
`1025 = 4 * 256 + 1` the number 1024 can be represented with 2 bytes with
values: 4 and 1. But how exactly do we write them? Is it `\x04\x01` or
`\x01\x04`? It occurs that both is possible and this is what
[*Endianness*](http://en.wikipedia.org/wiki/Endianness) is all about. The first
variant `\x04\x01` is called *big-endian*; it is used in Motorola 68000 and
PowerPC and some network protocols. The second variant `\x01\x04` is called
*little-endian* and is used in Intel x86 and AMD64.

Now stop for a second and think, how data is stored in memory or on disk? Yes,
that's it. Usually it happens in native binary format, which is what processor
operates on. And when data is sent over network it may be not in the native
order, but rather in order dictated by the protocol. So your Intel Core i7
processor will take the number `1025` as `\x01\x04` and reorder bytes to
`\x04\x01` before using it for a Total Length field in IP datagram and sending
it over Internet. As a side note, such order is specified by
[RFC 791](http://tools.ietf.org/html/rfc791#page-39) from 1981, but not by
earlier [RFC 760](http://tools.ietf.org/html/rfc760) from 1980.

The very same ideas apply when working with larger number of bytes, like 32-bit
or 64-bit integer.

### Alighment

Now, that we are mostly clear with primitive data types, let's talk about
structs. While computer addresses bytes, the operations are usually performed
on larger chunkgs. 64-bit processor natively summarize 8-byte numbers. SSE2
instruction takes 128-bit as operands. Cache blocks can be 64-byte long. From
all these examples it should be logical, that data should be optimized for
such things. For a struct of 1-byte character and 2-byte "short" integer one
may allocate 3 butes. But those 3 bytes should play well with 4-byte memory
access command. In particular if our struct is stored in array of such.
So compiler *pad* 3-bytes to 4 by adding one non-mininfull byte. And similarly
the "short" integer in struct will come to even memory address thus
faciliating some operations.

All these is called *alignment*. And this is the reason why bytes can appear
in different locations in memory or in binary format.

### Character

This is arguably the most difficult concept of all discussed here. Probably the
reason is the illusory simplicity. Indeed, looking at `A` letter we know that it
maps to decimal code `65`. Every even non-american keyboard has such a key and
consoles knowns how to depict this letter from `65` code in memory or read from
a file. Why to complicate simple things?

And indeed mainstream computer languages before 90's didn't make much deal of
characters and used a byte for one and bytearray for strings. How do they call
it - [ASCII](http://en.wikipedia.org/wiki/ASCII). But wait, ASCII encodes only
128 characters, not 256 and you have other languages than English. Hmmm. Yes,
now we mostly work with [Unicode](http://en.wikipedia.org/wiki/Unicode), which
cover most alphabets and other symbols, latest versions more than 110,000. And
this number is far beyond ability of one single byte.


