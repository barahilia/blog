---
layout: post
title:  "Points on plane compression"
date:   2014-10-11 20:58:00
categories: computers
---

<span style="background: yellow">
**TODO:**
</span>

* Background - v
* Define the task
* List several approaches
* Specify 256-square and 16-square encodings
* Summarize - what's the recommended way

## Background

In our website we need capability to depict a "live" 2D points viewer. Probably
the best example would be [Google Maps](https://maps.google.com/), but
our control is much simpler. While it does need zoom-in and panning, it's mostly
presents only one kind of objects, like "homes" that are defined solely by their
2-dimensional coordinate `(x, y)`. But there are a couple of thousands of them.
10K - 100K is an actual range. For the rest of the post we'll assume 50K points
as a close to average number in that range.

Previously, for a rich client, the viewer was built with
[OpenGL](https://www.opengl.org). After move to web a nice and simple control
was developed on top of [canvas](www.w3schools.com/html/html5_canvas.asp)
element with normal JavaScript library running in Chrome with no extensions.
After some not too heavy optimizations it become very fluent and responsive in
terms of presentation, pan and zoom times.

And with all that speed, we suddenly became aware of communication times. The
rich client loaded all the data including the points at startup. But now loading
the web page with the viewer started taking dozen of seconds to half a minute.
One generally doesn't wait that long for a browser. Actually several factors
contribute to that time: read data from the database, encode in JSON, send
through HTTP and decode back. Data sent came to 20-25MB, as each point has more
fields in addition to coordinates. The issue was temporarily mitigated with
compression, and total bulk reduced to 2MB. But whether one do much better for
this specific kind of data?

## The task

Suggest optimal representation for 50K 2D points where coordinates in both axes
are in range \[-30,000 .. 30,000\] (here I cheat a little, see below why this
range is so convenient). The result should be generated on the server side,
parsed on the client side and used for the points viewer with an ability to pan
and zoom.

## Approaches

First of all, the normal or native representation might be a CSV file with two
comma-separated doubles in each line. That is something like:

    667.21173417,602.336126150
    940.5201417,476.760126150
    ...
    465.30588,188.11905

This comes to 2MB for 50K lines.

1. Probably the easiest compression might be a PNG file where all the points are
   already depicted. PNG format utilizes lossless compressions, is easy to
   generate, doesn't require parsing and can easily support pan and zoom. But
   the quality of an image quickly reduces with zooming. Actually, initially
   generated image can be made very large, which will allow for a better zooming
   with the cost of resulted file size. A pretty good-looking picture can fit to
   a 100KB file, which you can't really enlarge.
1. Double numbers are long in textual representation, while occupy only 8 bytes
   in binary format. Given that we may simply encode our number pairs in binary
   format without any delimiters: 8 bytes for the x coordinate, 8 bytes for y,
   and so on. In total 8 bytes * 2 coordinates * 50K points = 800KB.
1. Next, when presenting many points in wide rectangle, you don't really need
   the precision of double. Unless one comes to a very fine zooming, which we in
   our specific project don't need. So doubles can be simply replaced with
   integers and even - note the range! - to signed short integers of 2 bytes
   each. I'm using here some specifics related to the project, but this can be
   easily generalized. Simply rescale different set of points to fit into short
   numbers. In any case, such a precision is mostly enough for the modern
   screen resolutions. So now we stand at 2 bytes * 2 coordinates * 50K points =
   200KB. Not a bad progress. So far, so good!

<span style="background: yellow">
Fill details here...
</span>

