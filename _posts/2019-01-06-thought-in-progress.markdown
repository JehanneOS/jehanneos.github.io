---
layout: post
title: Thought in progress
public: true
author: giacomo
image: /graphic/screenshot-20180106.png
css: 20170106.css
---
2018 has gone and from my point of view, it has been a great year.

I've learnt the hard way, a lot of valuable lessons that broke some
fundamental assumptions of mine, freeing me from several misconceptions and
giving me a priviledged point of view upon important matters.

First I discovered, with much surprise after years of cynism, that
Free Software and Open Source software are **completely different things**.

Everything started when I accidentally discovered that 
[Harvey had completely removed my name from their codebase without removing my contributions](https://medium.com/@giacomo_59737/what-i-wish-i-knew-before-contributing-to-open-source-dd63acd20696).  
This was a blatant violation of the
[GNU GPL (version 2)](http://www.gnu.org/licenses/gpl-2.0.html)
from [a member of Software Freedom Conservancy](https://sfconservancy.org/).  
And the commits that removed my name were all from a very experienced Google employee.

Since I can't sue both Google and SFC (they do no Evil, after all!), I had
to surrender when they squashed my changes into large commits (that included
work from several authors) during a huge `git rebase` that pretended to
remove my work (but have left several of my contributions there).

The problem is very practical to me, since I reused my contributions to
Harvey in Jehanne, and I'm afraid of a Google's lawyer could one day decide
that I copied Harvey because of the changes I authored myself.

All this was a valuable **revelation**. It tought me that:

1. we can't trust Google's employees when they contribute to OSS.
2. we can't trust SFC either, as [Karen Sandler](http://punkrocklawyer.com/)
   explicitly excluded such kind of issues when Harvey joined SFC.
3. we can't trust GNU licenses, as it's violated by SFC members without any issue.
4. talking about FOSS or FLOSS ultimately marginalize hackers.

# Beyond Jehanne

Reflecting on Google, I realized that 

- their lobbying against AI regulation is one of the most dangerous attack
  to worldwide freedom right now
- they own Mozilla that don't give a shit about people privacy or security

In fact the [Russian Government](https://bugzilla.mozilla.org/show_bug.cgi?id=1487081#c16)
is exploiting [one of the attacks](https://dev.to/shamar/the-meltdown-of-the-web-4p1m)
that I explained to [Mozilla](https://bugzilla.mozilla.org/show_bug.cgi?id=1487081)
and [Chromium](https://bugs.chromium.org/p/chromium/issues/detail?id=879381)
but the bug report is still "RESOLVED INVALID" because
"[this is the Web functioning as designed](https://bugzilla.mozilla.org/show_bug.cgi?id=1487081#c10)"

I mean the Russian Governement building a database of security researcher
and privacy aware people that could detect a JavaScript attack... is how
they want the Web to work. 

The worse vulnerability here is people trust in IT, which is broken beyond repair.

## The only mitigation is Culture

In 2018 I've spent a lot of my free time informing people about the risks of
these attacks. As a consequence, I was banned by a couple of online communities,
[Lobste.rs](https://dev.to/shamar/i-have-been-banned-from-lobsters-ask-me-anything-5041) and
[Tildes.net](http://www.tesio.it/cache/2018/TildesNET_Undetectable-Remote-Arbitrary-Code-Execution-Attacks-through-JavaScript-and-HTTP-headers-trickery.html).

Much better feedbacks came from [my talks about AI](http://www.tesio.it/talks/).

But ultimately, I realized that our only hope are kids and I started an
course of Informatics for the elementary school of my eldest daughter.

# What about Jehanne?

All these facts made me realize that existing copylefts cannot serve
hackers' curiosity, so I studied Copyright regulations to mitigate
the issues I saw with them. The result is the [Hacking License](http://www.tesio.it/documents/HACK.txt)
that is going to replace AGPLv3 as my copyleft of choice.

Writing the Hacking License took roughly 6 months.  
It's designed to protect users from SaaS's abuses by granting them the
ultimate right to self-host the application they use. This right will be
a great incentive to fair playing for SaaS providers using my File Protocol.
Also, the Hacking License is designed to turn users to hackers: IMO, this is
the only way Free Software can win against big corporations.

The few remaining free hours were spent into the design of the File
Protocol (aka FP) and the study of its integration in Jehanne as a
replacement of 9P2000.

At a first glance all you need to change to introduce an alternative file
protocol in a Plan 9 kernel is devmnt (in Jehanne it was already renamed as
[dev9p](https://github.com/JehanneOS/jehanne/blob/93111d7b0ea5b2b1582af1a576ccace7e1fcb9a3/sys/src/kern/port/dev9p.c))
but after a deeper analysis, it turns out that I have to modify the 
[Dev structure](https://github.com/JehanneOS/jehanne/blob/93111d7b0ea5b2b1582af1a576ccace7e1fcb9a3/sys/src/kern/port/portdat.h#L197)
because FP doesn't have a [Walk](http://man.cat-v.org/9front/5/walk) equivalent.

Changing `Dev` has a huge impact on the whole kernel, and it's just a part
of the issue because I also want to move to a [new mount table](http://lsub.org/export/9pix.pdf).

So ultimately, the Jehanne kernel is badly broken.

As a consequence you didn't saw much work ongoing on Jehanne, and to be honest... 
I wouldn't hold my breath.

The File Protocol is the reason Jehanne exists: I want to build the
infrastructure that will let people **hack** a **free** and **safe**
distributed system, so I have to address both the technical and the legal
issues to ensure it won't end like HTTP and browsers.

Because Jehanne is Free Software, **not** Open Source.
