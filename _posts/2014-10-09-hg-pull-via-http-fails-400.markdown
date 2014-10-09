---
layout: post
title:  "hg pull via HTTP fails with 400 error code"
date:   2014-10-09 18:14:00
categories: computers
---

It was Sunday evening. Coworker came and told me: "Team City fails; it can't
access repository. Can you possibly look at this?" Actually, then it was time
for me to get home. So I examined the build output briefly, understood it's not
for a couple of seconds and suggested to fix it tomorrow.

The error was like:

    Cannot collect changes. abort: HTTP Error 400: Bad Request

And on morrow that I started investigating this. The problem didn't reproduce
on my computer, so I assumed it's an internal TeamCity problem and tried to
workaround it, like removing caches or recreating the repository or using
different protocols. But soon the very same problem started happen to all of us.
We were unable to `hg pull` from our project repository.

