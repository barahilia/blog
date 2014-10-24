---
layout: post
title:  "Points on plane compression"
date:   2014-10-11 20:58:00
categories: computers
---

## Background

In our website we need a capability to depict "live" 2D points viewer. Probably
the best example would be the [Google Maps](https://maps.google.com/), but our
control is much simpler. While it does need zoom in and panning, it's mostly
presents only one kind of objects, like "homes" whose appearance is defined
solely by their 2-dimensional coordinate `(x, y)`. But there are a couple of
thousands of them. 10K - 100K is an actual range. For the rest of the post we'll
assume 50K points as a close to the average number in that range.

Previously, for a rich client, the viewer was built with
[OpenGL](https://www.opengl.org). After a move to web a nice and simple control
was developed on top of [canvas](http://www.w3schools.com/html/html5_canvas.asp)
element powered by a standard JavaScript library running in Chrome with no
extensions. After some not too heavy optimizations it became very fluent and
responsive in terms of presentation, pan and zoom times.

And with all that speed, we suddenly became aware of communication times. The
rich client loaded all the data including the points at startup. But now loading
the web page with the viewer started taking dozens of seconds. One generally
doesn't wait that much for a browser. Actually several factors contribute to
that time: read data from the database, encode it in JSON, send it through HTTP
and decode back. Data sent came to 20-25MB, as each point has some fields in
addition to coordinates. The issue was temporarily mitigated with compression,
and total bulk was reduced to 2MB. But whether one can do much better for this
specific kind of data?

## The task

Suggest optimal representation for 50K 2D points where coordinates in both axes
are in range \[-30,000 .. 30,000\] (here I cheat a little, see below why this
range is so convenient). The result should be generated on the server side,
parsed on the client side and used for the points viewer with an ability to pan
and zoom. The solution should allow for the viewer startup time to be as fast as
possible.

## Approaches

First of all, the normal or native representation might be a CSV file with two
comma-separated doubles in each line. Something like:

    6672.1173417,6023.36126150
    940.5201417,4767.60126150
    ...
    4653.0588,1881.1905

This comes to about 1.5MB for 50K lines and can serve a baseline.

### Image

Probably the easiest compression might be a PNG file where all the points are
already depicted. PNG format utilizes lossless compressions, is easy to
generate, doesn't require parsing and can easily support pan and zoom.
Unfortunately the quality of an image quickly reduces with zooming. Actually,
generated image can be made very large, which will allow for a better zooming
but at the cost of increased file size. A pretty good-looking picture can fit to
a 100KB file, which you can't really enlarge.

### Binary format

Double numbers are long in textual representation, while occupying only 8 bytes
in the binary format. Given that we may simply output number pairs without any
delimiters: 8 bytes for the x coordinate, 8 bytes for y, and so on. In total
8 bytes * 2 coordinates * 50K points = 800KB.

### Approximate doubles with ints

Next, when presenting many points in wide rectangle, you don't really need
the precision of double. Unless one comes to a very fine zooming, which we, in
our specific project, don't need. So doubles can be simply replaced with
integers and even - note the range! - to signed short integers of 2 bytes
each. I'm using here some details related to the project, but this can be
easily generalized. Simply rescale different set of points to fit into short
numbers. In any case, such a precision is mostly enough for the modern
screen resolutions. So now we stand at 2 bytes * 2 coordinates * 50K points =
200KB. Not a bad progress. So far, so good!

### Split to 256x256 squares

Is it possible to have only one byte for coordinate? Meaning, reduce to a
range \[0 .. 255\]. For this we should split the plane to 256x256 squares.
Assuming our points doesn't require the entire range \[-30000 .. 30000\] but
crowd inside \[-1280 .. 1280\] then they fit precisely into 100 sqaures of
256x256 each. Inside each such square we need only one byte per coordinate.
And to define it one needs a header of: `(X, Y)` for the square corner and
the number of points in this square to specify region in the output stream. Then
array of twice the number of points bytes will follow, like this:

    X,  Y,  n:  x1, y1; x2, y2; ...; xn, yn
    2B  2B  2B  1B  1B  1B  1B       1B  1B

    X,  Y,  n:  x1, y1; x2, ...
    2B  2B  2B  1B  1B  1B

The number of points in one square is less then 50K, hence fit nicely into
two bytes. In total we have 100 squares * 6 bytes per square header = 600B.
Then all the points together require 1B * 2 coordinates * 50K points = 100KB
which together with headers gives 101KB.

The encoding part was implemented and can be found in this
[Gist](https://gist.github.com/barahilia/0b05006c8e33f453e4f4).

### Split to 16x16 sub-squares

Can we reduce even further? Half a byte for coordinate, a byte for entire 2D
point, down towards \[0 .. 15\] range. Let's assume distribution of points
that is close to uniform. Then 50K points split to about 500 in every 256x256
square. Inside each there are 256 16x16 sub-squares with 2-3 points in every
sub-square. Per near uniformity assumption, we may assume each sub-square to
be occupied, hence one can write each of them them in some order, e.g. from
left to right, from bottom to top. And the only header needed is for the number
of points in the sub-square, like this:

    (0, 0) n:  x1, y1; x2, y2; ...; xn, yn
           4b  4b  4b  4b  4b       4b  4b
    (0,16) n:  ...
    ...
    (0,32) n:  ...
    ...
    (240,240) n:  ...

where `4b` stands for 4 bits or half a byte. Now there's 100 squares * 256
sub-squares in each * 0.5B per sub-square header = 13KB. In addition to 1KB
for square headers and 100KB / 2 = 50 KB for points themselves it leaves us
with bare 64KB in total!!

### Sort then compress

It can be noticed, that eventually, the ability to save less bits per each
single points was achieved due to organization of entire set in smaller units
of smaller area. And this is the essence of
[Huffman code](http://en.wikipedia.org/wiki/Huffman_coding) on which popular
achiving algorithms are based. So probably we don't need any code of our own?
Maybe usual compressing programs will do? And indeed, take the sample file
[`ints.csv`](https://gist.github.com/barahilia/0b05006c8e33f453e4f4#file-ints-csv)
with 50K lines of size 500KB looking like:

    -960,1619
    2592,805
    ...
    4382,3104

Next sort it and zip using maximal compression level:

    zip -9 ints.zip ints.csv                    # 231KB
    7zr a ints.7z ints.csv                      # 210KB
    
    cat ints.csv | sort -n > ints.sort.csv      # 516KB
    
    zip -9 ints.sort.zip ints.sort.csv          # 188KB
    7zr a ints.sort.7z ints.sort.csv            # 126KB

Meaning, `7-zip` indeed succeeds to utilize sorted data and to reach results
comparable with our 256 squares encoding - 126KB. One may assume even better
results if the file is arranged in the same way used by encodings: arrage
that all points that belong to the same 256x256 square adjust one another.
But experiment shows the opposite - archive size grows to about 150KB.

### An idea: describe with "patterns"

Completely different approach will be to discover "patterns" in the points and
to describe them in a short manner. Here probably even more precision might be
lost, but a more compact encoding can be achived. And probably the decoded
picture will look very similar to the real view and be easily resizable. In any
event this is just an idea and wasn't thoroughly checked yet.

## Summary

Apparently the optimal solution that require minimal investment and reaching
good compression level would be the "sort and zip". Every modern browser
supports compression, through `Content-Encoding: gzip` HTTP header and this is
seamless for the application on the client side. So virtually the only thing
needed for implementation is to sort the data. To make things even more faster
a precompressed data can be saved in advance in the database.


