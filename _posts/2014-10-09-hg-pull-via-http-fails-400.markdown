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
Hence a natural solution:
**enlarge allowed size of HTTP headers on the server side**.

And indeed
<http://mercurial.selenic.com/wiki/HgWebInIisOnWindows#Troubleshooting>
recommends the very same thing. And also presents the IIS defaults for the
maximal header size - 16KB or 16384 bytes. And our 16504 come just a few beyond
that.

Looks like the problem is resolved. But another question arises - why. There
are many repositories for other projects in our company served by the same
server. Ours is not the largest by any means. More than that, Mercurial has a
huge user base. Are we alone in our problem? And here are some:
<https://bitbucket.org/site/master/issue/8263/http-400-bad-request-error-when-pulling>,
<http://mercurial.808500.n3.nabble.com/Problem-pulling-through-http-proxy-td839537.html>.
There's even one very relevant bug on Mercurial:
<http://bz.selenic.com/show_bug.cgi?id=3319>.

After reviewing all those pages, the root cause becomes clear. Mercurial sees
the repository as [DAG](http://en.wikipedia.org/wiki/Directed_acyclic_graph) of
changesets taking into account the changeset itself, its parent(s) (generally
one, two in case of merges) and children. Branches, tags and other things come
as a filtering aid. In order to client to tell the server its status, it sends
hashes for the topological heads in DAG - list of all the changesets that have
no children, the most recent nodes in each line, ignoring any branches. The more
such heads there are, the bigger message to server.

Usually, there should be only one "main" head on the `default` branch, and
probably a couple of others, while people develop in parallel or work on
branches. And finally all parallel lines and branches merge into the mainstream.
And here comes the caveat, initially found in bug discussions for Bitbucket
and Mercurial (see the links above). When you merge two lines in `default`, two
heads become one. But when you merge branch back to `default`, Mercurial expects
you to close the branch. And you can do that in the following ways:

    close then merge                merge then close
    ----------------                ----------------
    
    @    default, head              @    default, head
    |                               |
    o    merge                      | x  close branch, head
    |\                              | |
    | x  close branch               o |  merge
    | |                             |\|
    o |  dev on default             o |  dev on default
    | |                             | |
    | o  dev on branch              | o  dev on branch
    | |                             | |
    | o  open branch                | o  open branch
    |/                              |/
    o    default                    o    default

Until this week both approaches look they same to me. It's more of aestetic
how exatly the branch is closed. Some advocate that if you merge first, then you
can close at some later stage after making sure the feature is completed. It can
be answered that merge should happen only after the feature is completed and any
further changes to be done in different branch. But for mercurial the difference
is essential: the left leaves only one head while the right leaves two of them.
And the second will forever participate in communications to the server
degrading performance and offensing scalability. So from now on, I'll always
recommend
**first to close the branch in mercurial and only then to merge it back to default**.

Guildeline for the case of completed features is clear. But what about the other
cases? Like:

* Discarded branches, that are closed and not merged into default
* Release branches, that are being maintained in parallel to default
* Other scenarios

The goal should be minimizing the number of topological heads. As suggested in
discussion of issue in Bitbucket, all discarded branches can be merged into one.
Same can be done to all release branches: once they come to their end of life,
merge them all into one dummy release branch. It should be remembered, that one
can continue working on a closed branch and even close it again.

Basically, the story ends here. For the curious, we currently stand at about
250 branches closed and opened, and the problem arose at about 240 heads - most
from branches that were merged first and closed after that.

