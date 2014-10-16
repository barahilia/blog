---
layout: post
title:  "It's all bytes..."
date:   2014-10-05 22:24:00
categories: encoding serialization endianity
---
We all know for ages, that computers work with bytes, normally, a byte comprises 8 bits and can take value from 0 to 255. There are 128 characters in basic ASCII encoding and this fit precisely into one byte even leaving room for localization. And `int`, which is generally 32-bit number, well... takes 4 bytes. Trivial things? For anybody in computer engineering indeed. But then you come for some specific task and thinks to yourself: "What on earth is going on?!"

Let's us start with some simple example: write a 32-bit integer to a file in binary format, as 4 bytes and read it back.

**TODO**:

* write a quick sample in C
* explain that in C strings are simply 0-ended byte arrays; no real separation between the char and byte
* show that in Python there's such a separation; default string is ASCII-based chars differ from bytearray
* show that in C# string is already Unicode by default
* explain how UTF-8 works; BOM

