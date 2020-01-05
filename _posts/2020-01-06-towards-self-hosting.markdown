---
layout: post
title: Towards self hosting
public: true
author: giacomo
---

Here we are, another year is gone.

# Ekaitz on board

The most important news for Jehanne in 2019, has been Ekaitz joining the development.

Ekaitz is a hacker and [an entrepreneur](https://elenq.tech/en/) and he is working to [port Jehanne's development tools](https://github.com/JehanneOS/devtools/commits/chibi) from Go to [Chibi-Scheme](http://synthcode.com/wiki/chibi-scheme).

Even if I don't exclude a Go port to Jehanne in the far future, this is important to reduce the complexity of the base operating system. Having the development tools in Chibi will reduce the effort to be totally self-hosted when the GCC port will be ready (hopefully soon).

# GCC 9.2.0

While having another hacker in the crew is great for Jehanne, during 2019 my work slowed down a lot.  

For several months I've been an active member of the Italian Pirate Party, just to realize that there is no space for hackers there (or for anybody who care about freedom and knowledge). Unfortunately learning this lesson required a huge waste of energy and free time.

In the remaining time I worked to port GCC 9.2.0 to Jehanne and, as this screenshot shows, I'm almost there.

<a href="/graphic/screenshot-20200106.png" target="_blank" title="GCC 9.2.0 running on Jehanne, click to enlarge">
	<img src="/graphic/screenshot-20200106.png" style="width: 600px"/>
</a>

As you can see, GCC can compile C programs on Jehanne.  
However I'm still having issues with C++ programs (like GCC itself, WTF!) so I didn't published the port yet.

It's not such an achievement, but running GCC on Jehanne is an first step towards self-hosting, proving the quality of the LibPOSIX approach. It also forced me to face and fix some of the errors I did in early stages of Jehanne development. I'll wrote more about such errors when the port will be complete, but the most evident ones have been a complete revision of user-space system calls, the rename of libc to libjehanne (to free `libc` for standard compliant C libraries) and a complete rewrite of the cross-compiler generation (moved outside the source tree).

There is a lot more work to do to reach self-hosting, but... this is a [long-term hack](https://github.com/JehanneOS/jehanne/blob/master/POLITICS.md).

