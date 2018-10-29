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

Sleep is a very interesting syscall because its whole point is just to
yield the CPU for a while, so that other processes can use it: it's
basically a tool for cooperative multitasking in a OS supporting
preemptive multitasking.

Other blocking system calls (like wait, read and write) depends on an
external, not completely predictable,, event to return control to the
process. Soon programmers realized that "not completely predictable"
can also mean "it could never happen" and looked for ways to ask the
CPU back after a while.

To this aim Unix introduced signals and services like
[alarm](http://pubs.opengroup.org/onlinepubs/9699919799/functions/alarm.html) and
[setitimer](http://pubs.opengroup.org/onlinepubs/9699919799/functions/setitimer.html)
to give back control to the calling process on certain events or after
a time out. 
