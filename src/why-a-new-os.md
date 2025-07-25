# Why a New OS?

The essential goal of the Redox project is to build a robust, reliable and safe general-purpose operating system. To that end, the following key design choices have been made.

## Written in Rust

Wherever possible, Redox code is written in [Rust](https://www.rust-lang.org/). Rust enforces a set of rules and checks on the use, sharing and deallocation of memory references. This almost entirely eliminates the potential for memory leaks, buffer overruns, use after free, and other [memory errors](https://en.wikipedia.org/wiki/Memory_safety#Types_of_memory_errors) that arise during development. The vast majority of security vulnerabilities in operating systems originate from memory errors. The Rust compiler prevents this type of error before the developer attempts to add it to the code base.

### Benefits

The following items summarize the Rust benefits:

- Memory-Safety

  All memory allocations are verified by the compiler to prevent bugs.

- Thread-Safety

  Concurrent code in programs is immune to data races.

- NULL-Safety

  NULLs can't cause undefined behavior.

## Microkernel Architecture

The [Microkernel Architecture](https://en.wikipedia.org/wiki/Microkernel) moves as much components as possible out of the operating system kernel. Drivers, subsystems and other operating system functionality run as independent processes on user-space (daemons). The kernel's main responsibility is the coordination of these processes, and the management of system resources to the processes.

Most kernels, other than some real-time operating systems, use an event-handler design. Hardware interrupts and application system calls, each one trigger an event invoking the appropriate handler. The kernel runs in supervisor-mode, with access to all the system's resources. In [Monolithic Kernels](https://en.wikipedia.org/wiki/Monolithic_kernel), the operating system's entire response to an event must be completed in supervisor mode. An error in the kernel, or even a misbehaving piece of hardware, can cause the system to enter a state where it is unable to respond to *any* event. And because of the large amount of code in the kernel, the potential for vulnerabilities while in supervisor mode is vastly greater than for a microkernel design.

In Redox, drivers and many system services can run in user-mode, similar to user programs, and the system can restrict them so they can only access the resources that they require for their designated purpose. If a driver fails or panics, it can be ignored or restarted with no impact on the rest of the system. A misbehaving piece of hardware might impact system performance or cause the loss of a service, but the kernel will continue to function and to provide whatever services remain available.

Thus Redox is an unique opportunity to show the microkernel potential for the mainstream operating systems universe.

### Benefits

The following items summarize the microkernel benefits:

- True modularity

  You can enable/disable/update most system components without a system restart, similar to but safer than some modules on monolithic kernels and [livepatching](https://en.wikipedia.org/wiki/Kpatch).

- Bug isolation

  Most system components run in user-space on a microkernel system. Because of this some types of bugs in most system components won't [crash or damage the system or kernel](https://en.wikipedia.org/wiki/Kernel_panic).

- More stable long execution

  When an operating system is left running for a long time (days, months or even years) it will activate many bugs and it's hard to know when they were activated, at some point these bugs can cause data corruption or crash the system.

  In a microkernel most system components are isolated and some bug types can't spread to other system components, thus the long execution tend to enable less bugs reducing the data corruption and downtime on servers.

  Also some system components can be restarted on-the-fly (without a full system restart) to disable the bugs of a long execution.

- Restartless design

  A mature microkernel changes very little (except for bug fixes), so you won't need to restart your system very often to update it.

  Since most of the system components are in userspace they can be restarted/updated on-the-fly, reducing the downtime of servers a lot.

- More stable and secure

  The system is more stable and secure because the kernel is much more simple (contain a very small amount of code), reducing the severity and impact of bugs.

- Easy to develop and debug

  Most of the system components run in userspace, simplifying the testing and debugging.

You can read more about the above benefits on the [Microkernels](./microkernels.md) page.

## Advanced Filesystem

Redox provides an advanced filesystem, [RedoxFS](https://gitlab.redox-os.org/redox-os/redoxfs). It includes many of the features in [ZFS](https://en.wikipedia.org/wiki/OpenZFS), but in a more modular design.

More details on RedoxFS can be found on the [RedoxFS](./redoxfs.md) page.

## Unix-like Tools and API

Redox provides a Unix-like command interface, with many everyday tools written in Rust but with familiar names and options. As well, Redox system services include a programming interface that is a subset of the [POSIX](https://en.wikipedia.org/wiki/POSIX) API, via [relibc](https://gitlab.redox-os.org/redox-os/relibc). This means that many Linux/POSIX programs can run on Redox with only recompilation. While the Redox team has a strong preference for having essential programs written in Rust, we are agnostic about the programming language for programs of the user's choice. This means an easy migration path for systems and programs previously developed for a Unix-like platform.
