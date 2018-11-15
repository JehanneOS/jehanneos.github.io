---
layout: post
title: Simplicity awakes
public: true
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
and designed ways to mitigate the risks.

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

# The Curse of Frankenstein

The [complex interactions](https://sourceware.org/bugzilla/show_bug.cgi?id=15819)
between the different ways to handle timeouts in Unix (and, to a
lesser extent, in Plan 9) are direct effects of its
[design philosophy](http://marmaro.de/docs/studium/unix-phil/unix-phil.pdf).

As explained by Richard P. Gabriel in
[his famous essay](https://www.dreamsongs.com/RiseOfWorseIsBetter.html),
"Unix and C are the ultimate computer viruses".

## Worse is better, isn't it?

Worse-is-Better is a model of software design and implementation which
has the following characteristics (in approximately descending order of
importance):

- **Simplicity**  
  The design must be simple, both in implementation and interface.
  It is more important for the implementation to be simple than the
  interface. Simplicity is the most important consideration in a design.
- **Correctness**  
  The design should be correct in all observable aspects, but it is
  slightly better to be simple than correct.
- **Consistency**  
  The design must not be overly inconsistent. Consistency can be
  sacrificed for simplicity in some cases, but it is better to drop
  those parts of the design that deal with less common circumstances
  than to introduce either complexity or inconsistency in the implementation.
- **Completeness**  
  The design must cover as many important situations as is practical.
  All reasonably expected cases should be covered.

Completeness can be sacrificed in favor of any other quality.
In fact, completeness must be sacrificed whenever implementation
simplicity is jeopardized. Consistency can be sacrificed to achieve
completeness if simplicity is retained; especially worthless is
consistency of interface.

In a way the so called "New Jersey style" was a rush for a
[minimum viable product](https://en.wikipedia.org/wiki/Minimum_viable_product)
able to minimize the 
[time-to-market](https://en.wikipedia.org/wiki/Time_to_market) and to gain the
[first mover advantage](https://en.wikipedia.org/wiki/First-mover_advantage).

Quoting Gabriel

> Both early Unix and C compilers had simple structures, are easy to
> port, require few machine resources to run, and provide about 50%-80%
> of what you want from an operating system and programming language.
>
> [...]
>
> It is important to remember that the initial virus has to be basically
> good. If so, the viral spread is assured as long as it is portable.
> **Once the virus has spread**, there will be **pressure to improve it**,
> possibly by increasing its functionality **closer to 90%**, but 
> **users have already been conditioned to accept worse** than the right thing.

So far so good.

Early Unix, like Plan 9, didn't aim at perfection, but at serving the
majority of users' need well enough.

## But is "enough", enough?

The Unix philosophy spread beyond Unix, and it became the common wisdom
of software engineering so far.

For early Unix and Plan 9, enough is enough.
Look at 9front to see a modern system that follows this philosophy consistently.

There is a catch, however.  
Plan 9 (like early Unix), is a system evolving as a whole.
This is particularly visible in the [9front repository](http://code.9front.org/hg/plan9front/).
The whole system evolve together, all programs are modified consistently,
answering to the ever changing needs of [its users](http://cat-v.org/).

Plan 9 is **one** application of computers.
An operating system split into several executables, but still **one thing**.

So in its asymptotic path toward perfection, the whole system slowly
progress together. When it provides 90% of required functionality, it
really means 90%. Or 95%. Or 97%. Or 99%.

But what if we split the system into independent components and assign
them to different teams following the same philosophy?

This is what slowly happened to mainstream operating systems.

With their widespread adoption, the variety of people needs couldn't be
served anymore by a single company. Thus good APIs and development tools
provided a strategic advantage to build ecosystems of applications that,
by running on a certain OS, would have expanded its market share.

Unix was particularly well suited for this kind of **race for
application developers**.

However this process had a significant but overlooked drawback: the
running operating system lose its unity, its coherence.
The operating system became a collection of variegate software,
developed by a variety of teams, each with their own goals and taste
and set of best practices.

## More pieces, more cracks!

Take one application that works correctly 99% of times, and you will be 
fine 99% of times.

But what if you have 10 applications each working correctly 99% of times?

The probability that they work correctly together at each computer
clock is (99%)^10. That is: their composition will do what you expect 
roughly **90% of times**.

With such assumption, the probability of a system behaving correctly
goes down pretty quickly approaching the formula *p(n) = (0.99)&#8319;*

![Probability of whole system correctness.](/graphic/probability_of_correctness.png)

How many programs are you running right now? :-D

This is the curse of Frankenstein: more pieces, more cracks.

## Simplex Sigillum Veri

Plan 9 from Bell Labs followed the New Jersey style from the very
beginning and still does so in most if it's incarnations.

But what about Jehanne,
[from Meucci's laboratory](https://www.theguardian.com/world/2002/jun/17/humanities.internationaleducationnews)?

Turns out the design of Jehanne doesn't follow the New Jersey style.
I have no rush. Yet Jehanne doesn't follow the MIT/Stanford style either.

To follow Gabriel's scheme, we could say that Jehanne is based upon

- **Simplicity**  
  The design must be simple. Few simple, easy to learn and orthogonal
  abstractions should be able to describe any use case conceivable.
  If the implementation is difficult, the design is not simple enough.
- **Correctness**  
  The design should be correct in all observable aspects.
  Incorrectness is not allowed.
- **Consistency**  
  The design must not be inconsistent.
  Any inconsistency reveals a design problem: either a missing
  abstraction or abstractions that are not orthogonal enough.
- **Composability**  
  Completeness should naturally emerge as a (desirable) side effect.
  As such, it cannot be a goal: the design must cover as **few**
  important situations as practical and let the user compose the
  abstractions provided to build what he needs.

I call this set of principles *simplex sigillum veri*.

It's deeply inspired by the works of other Italian hackers,
such as [Antonio Meucci](https://en.wikipedia.org/wiki/Antonio_Meucci),
[Pier Giorgio Perotto](https://en.wikipedia.org/wiki/Programma_101),
[Guglielmo Marconi](https://en.wikipedia.org/wiki/Guglielmo_Marconi),
[Renzo Piano](https://en.wikipedia.org/wiki/Renzo_Piano) and
[Leonardo da Vinci](https://en.wikipedia.org/wiki/Leonardo_da_Vinci).

# The Awake system call

Plan 9 supports **39 system calls** (not counting obsolete ones).
Since some system calls can be expressed in term of the others, in
Jehanne I moved the duplicates in userspace.

The idea is that a smaller kernel surface makes it easier to get it
right and simple in the long run.

In the process of this clean up, I realized that `sleep`, `alarm` and
`tsemacquire` were somehow "ugly": I supposed that they were mixing
orthogonal issues and looked for the missing abstraction they were hiding.

This is how the `awake` syscall was born:

```
long awake(long milliseconds);
```

Awake is the complement of `sleep`: it **books a new time slice** in the future.
It's a fundamental building block that can be used to implement
other services in user space, like `sleep`, `alarm` and `tsemacquire`.

It interrupts a blocking syscall after a certain number of milliseconds.
On failure it returns 0. On success it returns a negative long that can
be used as an identifier to release the booked time slice.

On wakeup, no note or signal is sent to the process, the process' error
string left unchanged: the blocking syscall simply returns to the
process with a `~0ULL` result.

A process can register how many wakeups it want (within a global system cap)
and each wake up will have a chance to interrupt a system call.

Wakeups are booked in two distinct group that do not interact: normal
process execution and note handlers. Such groups are automatically reset
respectively on [exits](http://man.cat-v.org/9front/2/exits) and on
[noted](http://man.cat-v.org/9front/2/notify).


In libc,
[two very simple functions](https://github.com/JehanneOS/jehanne/blob/master/sys/src/lib/c/9sys/awakened.c)
wrap `awake` to enhance readability.

`Awakened` tells the calling process whether a certain wakeup already occurred.
`Forgivewkp` tells the kernel to remove a certain wakeup.

With these primitives it's trivial to move
[sleep](https://github.com/JehanneOS/jehanne/blob/master/sys/src/lib/c/9sys/sleep.c) and
[tsemaquire](https://github.com/JehanneOS/jehanne/blob/master/sys/src/lib/c/9sys/tsemacquire.c) to libc.

In particular, `tsemaquire` shows pretty well the simple idiom of awake:

```
	/* book a time slice in the future */
	long wkup = awake(ms);
	while(blocking_syscall() == -1){
		if(jehanne_awakened(wkup)){
			/* handle syscall timed out */
		}
		/* the syscall has been otherwise interrupted, you can
		 * - try again
		 * - fail with an error
		 * - do whatever fit
		 */
	}
	/* the syscall completed, release the booked time slice... */
	jehanne_forgivewkp(wkup);

	/* ...and enjoy */
```

(the attent reader will notice that
[`alarm` is still waiting to be moved to user space](https://github.com/JehanneOS/jehanne/issues/2)...
the fact is that it's too boring of a task!)

To be fair, `awake` was originally designed to interrupt `rendezvous` only,
to enable [timeouts support for QLock, RWLock and Rendez](https://github.com/JehanneOS/jehanne/blob/master/sys/src/lib/c/9sys/qlock.c)
in libc.

But from the very beginning it was clear that it could have been
generalized and composed with other system calls, to provide other services.

With the advent of [libposix](https://github.com/JehanneOS/jehanne/tree/master/sys/src/lib/posix),
I used `awake` to implement support for POSIX
[non-blocking I/O](https://github.com/JehanneOS/jehanne/blob/master/sys/src/lib/posix/files.c),
[signals](https://github.com/JehanneOS/jehanne/blob/master/sys/src/lib/posix/processes.c)
and [sigset waiters](https://github.com/JehanneOS/jehanne/blob/master/sys/src/lib/posix/sigsets.c).

Which in turn enabled the port of [newlib](http://jehanne.io/2018/01/06/jehanne-in-2017.html) and of
[MirBSD Korn Shell](http://www.mirbsd.org/permalinks/wlog-10_e20180415-tg.htm)
to Jehanne.

All this with more or less [600 lines of code](https://github.com/JehanneOS/jehanne/blob/master/sys/src/kern/port/awake.c).
