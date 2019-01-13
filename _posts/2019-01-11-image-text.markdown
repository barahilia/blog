---
layout: post
title:  "Reading text in images"
date:   2019-01-11 09:00:00
categories: computers
---

Imaging an image or a picture, some usual shot from your phone camera. You
simply woke down the street and saw something funny or important. Nowadays, one
doesn't keep a mental note of a colorfull event. The smart phone is always
their. You reach for the device, take a picture and send it... Or, no! Nobody
sends anything for a couple of years. You share it with your spouse... Well,
again! Of course, to be politically correct, I should call share-e
"significant one". Now everything sounds just right.

What I was telling, is that the picture might contain some written text or
numbers, like signboard, or license plate, or street name, or even screen of
another phone. How one can extract and recognize this text? Well, the Computer
Vision and OCR technology is very advanced now. Services and apps with those
capabilities are in plenty. Still I wanted to attempt this feat on my own and an
occasion presented itself in a hackaton about a year ago.

My collegue and I split the job, with me discovering all potential areas with
captions and labels and him, extracting those areas and passing them through
OCR. How those areas can be found? I can imagine two principle approaches.
First, in the classical way, we build up the solution. Assuming, caption is
rectangular and has a border, some CV algorithm can be used to detect lines of
that border, connect them back into a polyline. Another idea would be to train
some ML model to detect promising areas. I chose the first path.

After some reading, and searching, the following steps emerged:
1. render the picture black-white
2. detect all the borderline pixels
3. find all the straight lines
4. discover all the rectangles
