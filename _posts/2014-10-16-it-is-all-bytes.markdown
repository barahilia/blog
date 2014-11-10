---
layout: post
title:  "It's all bytes..."
date:   2014-10-16 15:03:00
categories: computers
---

Every developer knows that computers and programming languages operate on bytes.
Hard disks are measured in gigabytes, memory is addressed up to single byte,
double number takes 8 bytes and so on. Byte comprises 8 bits and can take value
from 0 to 255. But when it comes to some very specific questions and actions we
halt and ask Google. And then copy ready recipes from
[Stack Overflow](http://stackoverflow.com/) without really understanding all the
nuances. E.g.:

* How to read an entire file to a string?
* How primitive data types are stored in memory?
* Why binary data from one computer looks like gibberish on another?

Let's try to describe some basic terms and to see how one may express them in
C++, Python (2.x) and C#.

This post intends to shed some light on how data represented in bytes looks like
in different mediums. But how to make bytes of data? For this *encoding* is
used, and there's at least one for every type of data. Here this is assumed to
be known or some relevant details are explained on the way. As well in order to
keep the post shorter, some advanced aspects of discussed concepts are
deliberately ignored.

## Notation

In some cases
[hexadecimal C-style notation](http://en.wikipedia.org/wiki/Hexadecimal#Using_0.E2.80.939_and_A.E2.80.93F)
will be used for a byte value: `\x00`, `\x01`, ..., `\xff`. So a sequence of 3
bytes with decimal values 10, 16, 254 will be written as `\x0a\x10\xfe`. With
such a notation, non-printable characters in a string are represented as well as
byte-arrays.

## Basic terms

### Endianness

An unsigned "short" 16-bit integer requires 2 bytes. E.g. since
`1025 = 4 * 256 + 1` the number 1025 can be represented with 2 bytes with
values: 4 and 1. But how exactly do we write them? Is it `\x04\x01` or
`\x01\x04`? It occurs that both are possible and this is what
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
it over Internet. (As a side note, such order is specified by
[RFC 791](http://tools.ietf.org/html/rfc791#page-39) from 1981, but not by
earlier [RFC 760](http://tools.ietf.org/html/rfc760) from 1980.)

Similar ordering extends to larger types including 32-bit and 64-bit integers.

### Alignment

Let's move from primitive data types to structs. While computer addresses single
bytes, the operations are usually performed on larger chunks. 64-bit processor
natively adds 8-byte numbers. SSE2 instruction takes 128-bit as operands.
[Cache](http://en.wikipedia.org/wiki/CPU_cache) blocks can be 64-byte long. From
all these examples it should be logical, that data should be optimized for such
things. For a struct of 1-byte character and 2-byte "short" integer three bytes
are needed in total. But those 3 bytes should fit into 4-bytes chunk on which
memory access commands operate. Two memory access commands will be required to
load the entire struct of 3 bytes that cross 4-byte border. And this happens for
array of 3-byte structs squeezed together.

So compiler *pads* 3-bytes to 4 by adding one non-meaningful byte. And similarly
the "short" integer in struct will come to even memory address thus facilitating
other operations.

All those actions are called *alignment*. And this is the reason why bytes can
appear in "unexpected" locations in memory or in binary format.

### Character

This is arguably the most difficult concept of all discussed here. Probably the
reason is its illusory simplicity. Indeed, looking at `A` letter we know that it
maps to decimal code `65`. Every even non-american keyboard has such a key and
consoles knowns how to depict this letter from `65` code in memory or read from
a file. Why to complicate simple things?

And indeed mainstream computer languages before 90's did not make much deal of
characters and used a byte for one and bytearray for strings. How do they call
it - [ASCII](http://en.wikipedia.org/wiki/ASCII). But wait, ASCII encodes only
128 characters, not 256 and you have other languages than English. Hmmm. Yes,
now we mostly work with [Unicode](http://en.wikipedia.org/wiki/Unicode), which
cover most alphabets and other symbols, latest versions more than 110,000. And
this number is far beyond ability of one single byte.

The story begins with the term *charset* - or code page, or charmap. It enlists
characters in our domain and gives their numbers - encoding. Some characters,
like `\n` end-of-line are functional and not printable. Others even not used in
textual files at all. There are many extensions of ASCII encoding national
letters and symbols with numbers 128-255 giving perfect fit to one byte code.
But in case of Unicode domain is significantly larger and in general 4 bytes are
used. *Encodings* define how to save Unicode character in file.
[UTF-8](http://en.wikipedia.org/wiki/UTF-8) is the dominant encoding in
Internet.

## To code!

### C++

Let's start from a simple program:

    struct data {
        char c;
        int i;
    };

    int main() {
        int i = 1025; // = 256*4 + 1 = \x01\x04
        short s = 16;
        char c = 127;
        data d; d.c = 65; d.i = 31;

        return 0;
    }

Compile it with `g++ -g 1.cpp` and hit debugger `gdb a.out` and execute:

    (gdb) break 20          // set breakpoint at return statement
    (gdb) start             // run the program, stop at main entrance
    (gdb) continue          // run to breakpoint
    (gdb) print &i          // print memory address of the first variable
    $5 = (int *) 0xbffff054
    (gdb) x/24b 0xbffff050  // examine 24 bytes in memory starting at
    0xbffff050:     -60   c:127    s:16-------0     i:1-------4-------0-------0
    0xbffff058:  d.c:65    -121       4       8  d.i:31-------0-------0-------0

Look, how little-endian and alignment manifest themselves in this example. The
4-byte long integer numbers are placed at memory addresses that divide to 4
without remainder and there's pad of 3 bytes between `d.c` one byte and `d.i`.
You may also pay attention, that compiler reordered local variables but left
layout of struct members intact. This is normal behavior. In between random
padding bytes are left.

The `char` data type in C/C++ is one byte long. It is convenient for work with
ASCII files or direct map between memory, byte array and file content. And since
the string is essentially byte array, one may easily read entire file or its
part to the memory and use it as text or binary.

### Python 2.x

From "bytes" perspective, Python 2.x is very similar to C. Default strings are
also byte arrays. So one can read entire file with `s = open("file.ext").read()`
and process it as ASCII string or as binary data. Python is however more limited
in raw memory access. While in C every single byte is accessible with pointers,
Python is closer to human kind. It has dedicated
[struct](https://docs.python.org/2/library/struct.html) module packs and unpacks
bytes. See for example:

    from struct import pack, unpack

    p = pack("!ih", 65, 66)         # network order: integer, short
    open('data.dat', 'w').write(p)
    print repr(p)                   # '\x00\x00\x00A\x00B'

    s = open('data.dat').read()
    t = unpack("!ih", s)
    print t                         # (65, 66)

Note, that we specified the byte order - network or big-endian, so that this
file will gain the same results on all architectures. With the same ease, one
may do byte manipulations with this module. E.g. split 32-bit integer to two
16-bit shorts, just like in C.

Python 2.x also provides `unicode` data type for Unicode strings. Given file:

    aaa угу bbb

(note Russian letters in the middle) we may do:

    s = open('text.txt').read() # incorrectly reads as ASCII / byte array
    print repr(s)               # 'aaa \xd0\xb5\xd0\xbb\xd1\x8c bbb\n'
    
    import codecs               # encoder/decoder
    u = codecs.open("text.txt", "r", "utf-8").read()
    print repr(u)               # u'aaa \u0435\u043b\u044c bbb\n'

### C\#

C\# is a modern language. Being such it's fully aware of byte quirks and the
difference between character and byte. The
[char](http://msdn.microsoft.com/en-us/library/x9h8tsay.aspx) data type
explicitly represents a Unicode character. It is 16-bit long which is enough for
most usages. So while we're working with data, we have to tell .NET plainly that
bytes should be read or written:

    public static void Main()
    {
        int i = 65; short s = 66;
        var bytes = Enumerable.Concat(
            BitConverter.GetBytes(i),
            BitConverter.GetBytes(s)
        );        
        File.WriteAllBytes("bytes.bin", bytes.ToArray());
        
        byte[] read = File.ReadAllBytes("bytes.bin");
        Console.WriteLine(
            "Read: int {0} and short {1}",
            BitConverter.ToInt32(read, 0),
            BitConverter.ToInt16(read, 4)
        );
    }

And what will happen if textual file is read into a string? Let's try:

    public static void Main()
    {
        string s = File.ReadAllText("text.txt");
        Console.WriteLine(s); // aaa угу bbb
    }

[`File.ReadAllText()`](http://msdn.microsoft.com/en-us/library/ms143368(v=vs.110).aspx)
automatically detects UTF-8 encoding and loads file content correctly. C\# makes
clear distinction between the byte array the string.

## Bottom line

All data is made of bytes. To build basic types as integer, struct or string one
should be aware of *endianness*, *alignment* and *characters*. We understood how
to operate on those concepts in three mainstream programming languages C++,
Python and C\#.

