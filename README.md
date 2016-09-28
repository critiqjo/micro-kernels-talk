# Return of the Microkernels

## Motivation

Fields dominated by embedded and real-time systems (more specifically, high-assurance systems) such as avionics, medical and military systems, etc. where the reliability requirement is so high that “a failure could mean somebody dying,” primarily use microkernels.

It does not look like microkernels will become the default in general purpose computers, but more like systems with such reliability requirements will eventually become part of our everyday lives (such as autos, or networked IoT systems where security is very critical).

Linux performance over security (old example): vsyscall vs vdso (fixed in 2.6): http://stackoverflow.com/a/19942352/2849934

## Non-goals

I’m not too familiar with the problem and solution space other than a bit of exposure to some articles and talks, therefore a rigorous discussion on design/implementation specifics is outside the scope of this talk. I’m just here to claim that their design principles are still relevant today and will be increasingly so in the future (by pointing to ongoing works as indicators).

## Outline

_(blank)_

## Architecture Overview

We'll discuss the principles behind the microkernel design later, but here just notice that mono-kernels have a giant blob of code running in kernel mode.

## Mono-kernels

_(blank)_

## Microkernels

**Mechanism**: μ-kernel typically provides basic IPC, address space management mechanisms, hardware endpoint management, and process management and scheduling. But who gets access to which part of address space or device endpoint is managed by **policies** implemented on top of it.

E.g. Memory management (va -> pa mapping, what to do if memory is almost full etc.) can be done in by a user-mode server, (but still page tables (faults) should be handled by kernel)... A “path” in Linux may map to a file, a network socket etc. which is a policy. Reading data from disk is an underlying mechanism, done by the VFS through drivers.

**Fault isolation**: a lousy graphics driver can no longer take down the entire system or be an attack surface for a malware trying to take over the entire system. (fault recovery: restarting a crashed driver, but that alone won’t guarantee a safe recovery, which would need checkpointing)

**Extensibility**: Components can be updated in isolation (e.g. upgrade or add a new feature to VFS or MemMngr without recompiling the whole kernel)

**Responsiveness**: Since much more code is running in pre-emptible userspace, say, a blocking driver will not affect the system responsiveness. Better suited for real-time systems which might need (response) time-bound guarantees.

**EoC**: Defining clear boundaries between different parts of the system and limiting unwanted and unnecessary interaction (when everything is in one big blob, people tend to do dirty hacks to get things done faster which could become a maintenance pain)

**Context switches** are fine as long as cache misses are little affected. Note: disk access through IPC does not mean data is transferred through IPC. Bulk data is always transferred through shared pages (at least in well-designed ones).

## Context Switches

**The costs**: Aside from TLB flushing being an expensive operation, the va->pa translations takes more cycles

**Tagged TLB entries**: Translations can be tagged with a “process id” so that instead of flushing the TLB, simply consider an entry valid if the ids match

**Global pages**: Entries can be marked as “global” for kernel pages so that they are not flushed at all

**Large pages**: This reduces the number of entries needed per unit of memory

## Kid \#1: L4Linux/L4Re
L4 is a family of second-gen microkernels; originally a microkernel designed and implemented by Jochen Liedtke as a response to the poor performance of earlier microkernel-based operating systems.

Still active Linux 4.x versions have been released recently, but couldn’t find recent docs on design. Originally designed for real-time systems when Linux was not pre-emptible. But now, it is might be used as a virtualization solution?

## Kid \#2: Genode

Quotas: resource trading for deterministic behavior

## Kid \#3: HelenOS

The design docs is kinda outdated (v0.2 vs current v0.6), but most ideas are probably still valid.

Principles: Microkernel principle, Split of mechanism and policy, Multiserver principle, Full-fledged principle, Encapsulation principle, Portability principle

## Kid \#4: seL4

Capability-based security: where the entity requesting a service holds an unforgeable certificate of permission for access which is verified by the authority granting access.

Another microkernel which claims to be “first ever” formally verified: Muen Separation Kernel (https://muen.sk/) (also targeting high-assurance systems; eg. aircraft controllers)

## Kid \#4: seL4

Specifically designed with formal verification in mind. For more information, see [2.b]

## Kid \#5: Fuchsia/Magenta

Development started last June (according to the repo history).

IoT devices exposed to open network and running critical software such as, say, auto-driving program need very high security guarantees.

## Closing thoughts

_(blank)_

## Thank you!

The actual author of that quote is disputed! The oldest known evidence goes to Clare Booth Luce (http://quoteinvestigator.com/2015/04/02/simple/)
