---
layout: post
title: Awake: wakeups in Jehanne
public: false
author: giacomo
image: /graphic/screenshot-20180106.png
---

Preemptive multitasking is now more than 50 years old, initially
introduced in PDP-6 Monitor and MULTICS in 1964. Instead of relying on
processes to cooperatively release the CPU to the kernel, they
registered [timer interrupt handlers](https://en.wikipedia.org/wiki/Interrupt)
to move control back to kernel code.

Nevertheless, processes were still able to yield control of the CPU to
the kernel during a [time-slice](https://en.wikipedia.org/wiki/Preemption_(computing)#Time_slice)
by calling any [system call](https://en.wikipedia.org/wiki/System_call).

System calls are often distinguished into blocking or non-blocking
depending on the probability that the kernel will return the control
to the calling process' code before the next tick.

In real world, non-blocking syscalls might block the process and
blocking syscalls might not block at all: it all depends on
implementation details and on the state of the system at run-time.

Classical examples of blocking system calls are

- `sleep` that should block for a certain number of seconds
- `wait` that could block until a children process exit
- `read` that would block until a chunk of data is available
- `write` that would block on a pipe without enough space for the new data

## CPU control and time

Sleep is a very interesting syscall because its whole point is just to
yield the CPU for a while, so that other processes can use it.
Its main use case is to poll for a certain resource to become available
without subtracting computing power that could be used to produce such
resource: it's a tool for cooperative multitasking in a OS supporting
preemptive multitasking.

Other blocking system calls (like wait, read and write) depend on
external events to return control to the process.
Soon programmers realized that an external event might never occur
and designed ways to book a time slice in the future.

Unix introduced [signals](https://en.wikipedia.org/wiki/Signal_(IPC))
and services like
[alarm](http://pubs.opengroup.org/onlinepubs/9699919799/functions/alarm.html) and
[setitimer](http://pubs.opengroup.org/onlinepubs/9699919799/functions/setitimer.html)
to give back control to the calling process on certain events or after
a certain time.

Later, new syscalls like
[select](https://idea.popcount.org/2016-11-01-a-brief-history-of-select2/),
[poll](http://pubs.opengroup.org/onlinepubs/7908799/xsh/poll.html),
[kqueue](https://www.freebsd.org/cgi/man.cgi?query=kqueue&sektion=2) or
[sigtimedwait](http://pubs.opengroup.org/onlinepubs/9699919799/functions/sigtimedwait.html)
were introduced with a timeout parameter from the very begining.

## Plan 9 from Bell Labs

Compared to the [400 system calls](http://man7.org/linux/man-pages/man2/syscalls.2.html)
of Linux, Plan 9's [API is rather simpler](http://aiju.de/plan_9/plan9-syscalls)
but it still supports a bit of each time-control styles:

- [sleep](http://man.9front.org/2/sleep) suspends the calling process
  for a number of milliseconds specified by the argument.
- [alarm](http://man.9front.org/2/sleep) causes an
  [alarm note](http://man.9front.org/2/notify) to be sent to the
  invoking process after a number of milliseconds.
- [tsemacquire](http://man.9front.org/2/semacquire) only waits for a
  number of meilliseconds to acquire a semaphore.

By design, Plan 9 provides only a very limited support for
non-blocking I/O (through `alarm`) and no support for
[I/O multiplexing](https://notes.shichao.io/unp/ch6/):

- if a program needs to **wait for several resources**, it usually calls
  [rfork](http://man.9front.org/2/fork)
- if a program want to **serve several concurrent clients**, it usually
  expose a [9P2000 filesystem](http://man.9front.org/5/intro)

Furthermore, with [libthread](http://man.cat-v.org/9front/2/thread),
Plan 9 provides an implementation of Hoare's
[Comunicating Sequential Processes](http://www.usingcsp.com/cspbook.pdf)
in which dedicated (preemptively scheduled) processes are used to issue
any blocking system calls and several cooperatively-scheduled threads
sharing memory in a single process are used to access global state.

# Simplex Sigillum Veri

The [complex interactions](https://sourceware.org/bugzilla/show_bug.cgi?id=15819)
between the different time ways to book a time slice in Unix (and, to a
lesser extent, in Plan 9) are direct conseguences of its
[design philosophy](http://marmaro.de/docs/studium/unix-phil/unix-phil.pdf).

As explained by Richard P. Gabriel in
[his famous essay](https://www.dreamsongs.com/RiseOfWorseIsBetter.html),
"Unix and C are the ultimate computer viruses".
