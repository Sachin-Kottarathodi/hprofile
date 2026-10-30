---
title: "How Operating Systems and Hardware Enforce Atomicity"
date: 2026-04-14
draft: false
tags:
  - system-design
  - tech
series:
  - Locks
---

# From File Systems to Transistors: How Operating Systems and Hardware Enforce Atomicity - Part 1
## AI Generated

> **Problem Statement:** If two CPU cores attempt the same operation at
> almost exactly the same instant---such as creating `temp.txt` with
> `O_CREAT | O_EXCL`---why does only one succeed?

Every backend engineer has relied on file-system atomicity. Creating a
lock file with `open(..., O_CREAT | O_EXCL)` or using `mkdir()` as a
lock are common synchronization techniques.

If two processes attempt to create the same file at nearly the same
time, the operating system guarantees that exactly one succeeds while
the other receives `EEXIST`.

How does that guarantee emerge?

The answer spans the entire stack---from the Virtual File System (VFS),
through kernel synchronization, atomic CPU instructions, cache
coherence, and finally into the digital hardware that establishes a
single observable order for conflicting operations.

------------------------------------------------------------------------

## Layer 1: File System (VFS)

Both processes enter the kernel through a system call.

``` text
Process A ----\
               --> VFS --> Parent Directory --> Synchronization --> Update Directory
Process B ----/
```

Before modifying a directory, the kernel protects the directory's
metadata using kernel synchronization primitives. The exact primitive is
an implementation detail and may vary across kernel versions, but the
guarantee is the same:

-   one thread performs the conflicting modification,
-   others wait,
-   later arrivals observe the updated directory state.

The winner creates the directory entry.

The second process eventually examines the directory, finds that
`temp.txt` already exists, and returns `EEXIST`.

------------------------------------------------------------------------

## Layer 2: Kernel Synchronization

Kernel synchronization protects shared state.

Conceptually:

``` text
Shared State
     │
     ├── Thread A acquires ownership
     └── Thread B waits
```

Internally, Linux uses synchronization primitives such as mutexes,
reader-writer semaphores, spinlocks and wait queues depending on the
situation.

The important abstraction is **not which primitive is used**, but that
the kernel guarantees only one execution path modifies the shared data
at a time.

If another thread cannot proceed immediately, it is typically placed on
a wait queue and scheduled later instead of wasting CPU cycles.

------------------------------------------------------------------------

## Layer 3: Atomic CPU Operations

Eventually the kernel relies on atomic hardware operations.

On x86 this often involves instructions such as `CMPXCHG` together with
atomic memory semantics.

Other architectures provide equivalent primitives (for example ARM's
Load-Exclusive / Store-Exclusive instructions).

Different instruction sets use different mechanisms, but they all
provide the same guarantee:

> Read, modify and write shared memory as one indivisible operation.

------------------------------------------------------------------------

## Layer 4: Cache Coherence

Multiple CPU cores each have private caches.

If two cores both wish to modify the same memory location, they cannot
both own that cache line for writing simultaneously.

The processor's cache-coherence protocol establishes a globally
consistent order for conflicting ownership requests.

Conceptually:

``` text
Core 1  ----\
              ---> Coherence subsystem ---> Exclusive ownership granted
Core 2  ----/
```

One request is processed first.

The other observes the updated state and therefore fails its
compare-and-exchange operation.

The exact implementation differs across processors (Intel, AMD, ARM,
Apple Silicon, etc.), but the architectural guarantee remains the same.

------------------------------------------------------------------------

## Layer 5: Digital Hardware

At the lowest level, processors are built from clocked digital logic.

Hardware is designed so conflicting requests cannot both commit
simultaneously.

The processor establishes a deterministic ordering for writes to the
same shared memory location, allowing software to observe a single
consistent outcome.

The implementation involves complex hardware---cache controllers,
coherence logic, interconnects and digital state machines---but those
details are intentionally hidden behind the abstraction of atomic memory
operations.

------------------------------------------------------------------------

## Putting It Together

``` text
Application
    │
    ▼
File System
    │
    ▼
Kernel Synchronization
    │
    ▼
Atomic CPU Instruction
    │
    ▼
Cache Coherence
    │
    ▼
Digital Hardware
```

Each layer builds on guarantees provided by the layer below it.

The implementation changes.

The guarantee does not.

------------------------------------------------------------------------

## Conclusion

When you call:

``` c
open("lock.file", O_CREAT | O_EXCL)
```

you rely on an entire chain of synchronization.

The file system serializes conflicting directory updates.

The kernel protects shared state with synchronization primitives.

The processor performs atomic memory operations.

The cache-coherence protocol ensures every core agrees on the order of
conflicting writes.

Modern computers do **not** eliminate parallelism. Millions of
operations execute concurrently every second.

What they eliminate is **ambiguity**.

Whenever multiple actors compete to modify the same shared state, the
system guarantees one deterministic outcome that every participant
ultimately observes.



