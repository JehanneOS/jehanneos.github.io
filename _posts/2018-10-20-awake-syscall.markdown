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
the kernel by calling any [system call](https://en.wikipedia.org/wiki/System_call).
