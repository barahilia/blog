---
layout: post
title:  "Impossible Java"
date:   2017-03-26 18:40:00
categories: computers
---

## Mysterious snippet

Some time ago I came by a very strange piece of Java code. It was extracted from
a *working* application. In a simplified form it looks like this:

```java
class C {
    void a(String s) {}
    int a(String s) { return 0; }
}
```

Well, this is a clear syntactic error. And sure enough, `javac` tells:

```
C.java:3: error: method a(String) is already defined in class C
```

## Overloading functions

It cannot be otherwise. Any compiler of sound mind and memory will issue an
error. Not only for Java. Same thing happens in any statically-typed
object-oriented language allowing functions overloading in class: functions of
the same name and number of arguments must have different types of arguments.

This rule is explained in basic textbooks and programming courses. And the
reason is this: function invocation in such languages is usually an *expression*
and the compiler must choose appropriate implementation at compile-time. Meaning
the following code `a.b(c, d)` is regarded as a complete expression and the
compiler will decide which function to be called at this point.

To make the decision, our compiler will check the types of `a`, `c` and `d`. We
are talking about staticly-typed languages, Java included. All the type
information is declared and known at compile time. Then the matching `b(,)`
function will be chosen. Again, we are looking at expression and are aware of
parameters' types.

But the expected return type isn't known. On the contrary it is be inferred from
the chosen matching function. And if there is more than one option, the compiler
is at loss. This is the case in the example above. Both the first and the second
function accept the same one argument of type `String`. Which is the right one?

## A bit about Android app

How such a snippet could have possibly get into working application? It is
impossible. There is *no way to compile it*. So I thought. Then wiped my eyes.
And then - eureka! I actually wasn't looking at compilable code. But rather it
was *decompiled* from a virtual machine bytecode.

The application was an Android application. They are usually written in Java,
compiled and packed into APK files. When we install an app our smart phone
downloads the APK file, which is actually a ZIP archive with a specified
structure. There are a number of files inside like images, fonts, certificate.
One of files is called `classes.dex`. It is a DEX file containing the binary
code for Dalvik - the Android Java Virtual Machine.

There are a number of tools for converting DEX into human readable mnemonics,
just like Assembler, but more high-level. And there are also tools capable of
decompiling Dalvik opcodes back to Java. Frankly, the result may differ a lot
from the original source code, but semantically they should be equal. In the
past I worked with [IlSpy](http://ilspy.net/) - a decompiler for .NET. It had
achieved spectacular results. Sometimes the decompiled version was almost
indistinguishable from the origin.

In mnemonics our example looks this way:

```smali
# L and ; decorate the reference type name
.class LC;

.method a(Ljava/lang/String;)V  # V stands for "return type void"
    # statements...
.end method

.method a(Ljava/lang/String;)I  # I stands for "return type int"
    # statements...
.end method
```

## Method invocation

In order to call `C.a()` the following opcode will be used:

```smali
    invoke-direct {v0, v1}, LC;->a(Ljava/lang/String;)I
    #                       ^.........................^
```

And here is the clue: the entire signature is being used including the function
name, the argument types and *the return type*! For the reference I would
recommend to browse
[http://pallergabor.uw.hu/androidblog/dalvik_opcodes.html](http://pallergabor.uw.hu/androidblog/dalvik_opcodes.html)
and to search for `invoke-direct` opcode. You see the point? To Dalvik, there is
simply no such thing as a function name by its own. In order to declare or to
call a function entire signature, including the return type, must be specified.

Now the answer to "how is this possible," is clear. Decompiler simply built Java
code from Dalvik opcodes. And what was sane and executable to the Virtual
Machine suddenly presented itself as high level code that compiler rejects. The
next question is: how and why this has happened? We may assume that originally
there was valid Java code. How our example has come into being?

## Concealing the code

Like with assembler, one can write Dalvik mnemonics directly and produce machine
code without any aid from high level language compiler. This indeed happens for
various reasons. In Android world, much like .NET, Java, JavaScript and more,
there is one more player - obfuscator. Existence of decompilers is a problem:
once an app was developed and published, nothing prevents adversaries taking the
bytecode, putting it back to readable form and seeing what and how the app does.

One may want to guard its code and conceal how exactly it ticks.
[Obfuscators](https://en.wikipedia.org/wiki/ProGuard_(software)#Obfuscation)
try to help to some extent. The easiest thing they can do is replacing names of
all packages, classes and functions to some meaningless strings in a consistent
way. To Dalvik it does not matter. But whoever tries to read the code will need
to dig much deeper. E.g. `byte[] downloadFileFromUrl(String url)` can become
`byte[] a(String b)` or `double computeLoanInterest(int days)` can be renamed to
`double c(int d)`.

Apparently some clever obfuscator used the fact that Dalvik requires entire
function signature together with its name at method invocation. And so it took

```java
class C {
    void first(String s) {}
    int second(String s) { return 0; }
}
```

And instead of renaming `first` to `a` and `second` to `b` it renamed them both
to `a`, which is still valid in VM because of the different return types! That's
it. End of story. Dalvik happily executed the code. And Java decompiler
diligently translated methods to the high level language unaware of the
catch.

## Insane Java

Well, until now the snippet was simply impossible. Now, to raise the level, a
really insane thing is going to happen. Try to imagine a class with two
different fields with the same name:

```java
class C {
    int a;
    String a;
}
```

It was discovered in the same application. And again the clue comes from the
Dalvik opcodes reference. Let's say `v1` is of type `C` as above and we need to
read value of the second member `String a` into `v0` like: `v0 = v1.a`. The
following opcode will be used:

```smali
    iget v0, v1, LC;->a:Ljava/lang/String;
    #            ^.......................^
```

Again, the Dalvik syntax involves the fully resolved class name, the field
name and the field's type. No problem to have a dozen of fields with the same
name assuming all of them are of different types.
