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
registered [interrupt handlers](https://en.wikipedia.org/wiki/Interrupt)
to move control back to kernel code.
Timers' interrupts were then used to periodically
[switch CPU control](https://wiki.osdev.org/Context_Switching) among
several pending processes, improving the illusion of
[concurrent execution](https://wiki.osdev.org/Scheduling_Algorithms) on
single processor machines.

Nevertheless, processes were still able to yield control of the CPU to
the kernel during a [time-slice](https://en.wikipedia.org/wiki/Preemption_(computing)#Time_slice)
by calling any [system call](https://en.wikipedia.org/wiki/System_call).

System calls are often distinguished into blocking or non-blocking
depending on the probability that the kernel will return the control
to the calling process' code before the next tick.

In real world, non-blocking syscalls might block the process and
blocking syscalls might not block at all: it all depends on
implementation details and on the state of the system at run-time.

Classical examples of 
