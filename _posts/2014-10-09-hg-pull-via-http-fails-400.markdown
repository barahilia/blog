---
layout: post
title:  "hg pull via HTTP fails with 400 error code"
date:   2014-10-09 18:14:00
categories: computers
---

<span style="background: yellow">
**TODO:** since the post become so long, add an intro and summary sections.
</span>

It was Sunday evening. Coworker came and told me: "Team City fails; it can't
access repository. Can you possibly look at this?" Actually, then it was time
for me to go home. So I examined the build output briefly, understood it's not
for a couple of seconds and suggested to fix it tomorrow.

The error was like:

    Cannot collect changes.
    abort: HTTP Error 400: Bad Request

And on the morrow I started investigating this. The problem didn't reproduce
on my computer, so I assumed it's an internal TeamCity problem and tried to
workaround it, like removing caches or recreating the repository or using
different protocols. But soon the very same problem started happen to all of us.
We were unable to `hg pull` from our project repository.

At that point it's became clear, that the problem lies in the central
repository. Since `hg pull -v --debug` doesn't add much info we decide to
attempt inspection of the HTTP traffic to get more details.
Let's run [Wireshark](https://www.wireshark.org/). Fine, it grabs and depicts
all the network communication -
wow, it might come to dozens of packets in second for any usual computer. And
even after filtering the relevant data with filters like
`ip.dst == 192.168.1.1 and http` it's difficult to make anything from results.
[Fiddler](https://www.wireshark.org/) is much more convenient for HTTP. But -
alas, - the tool shows only
requests from browsers. Search instantly presents simple solution at
<https://groups.google.com/forum/#!topic/httpfiddler/ImX0g6ufdlM>:

    hg --config http_proxy.host=127.0.0.1:8888 pull

Meaning - mercurial will use port `8888` as a proxy. Phew, finally some first
results. A couple of request-response run and the last one completes indeed
with `400 bad request`. But now we have the entire response:

<span style="background: yellow">
**TODO:** insert here output from IIS - too long headers.
</span>

Here's the first culprit: too long headers. And indeed, the last request is
16504 bytes long. Quite a large one, you would expect more from Mercurial...
But after some thought it's clear, that the client should send the server
enough data for comparison between the two repositories, for server to send back
only the missing changesets. We're nearing respectful number of 5000 commits
which justifies need for larger request size. On the other hand, Mercurial
is an industrial level system and much larger projects should be supported.
Hence a natural solution: **enlarge allowed size of HTTP headers on the server
side**.

And indeed
<http://mercurial.selenic.com/wiki/HgWebInIisOnWindows#Troubleshooting>
recommends the very same thing.


