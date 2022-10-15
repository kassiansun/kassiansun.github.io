---
title: 'Go Runtime Implementations: Scheduling'
date: 2022-10-14T18:31:51+08:00
tags:
  - go
---

- Code Path: `src/runtime/proc.go`

## Start-Up Process of a Go program.

Take `src/runtime/asm_arm.s` as an example:

- Take the address of `g0` and `m0`.
- Set up `m.g0` and `g.m`.
- Create `istack`.
- Do runtime check.
- Save `argc` and `argv`.
- Call `runtime.osinit`.
- Call `runtime.schedinit`.
- Call `runtime.newproc` for the main function.
- call `runtime.mstart` to start the M.

### `runtime.osinit`

Take `src/runtime/os_linux.go` as an example:

- Get the number of processors. This is done by making a syscall `SYS_sched_getaffinity`.
- Get the huge page size.
- Run osArchInit. <- Seems not used.

### `runtime.schedinit`

- Initialize all the locks.
- Set up the `g.racectx`.
- Stop the world.
- `moduledataverify`. Defined in `src/runtime/symtab.go`, it will check `moduledata`'s binary info.
- `stackinit`. Defined in `src/runtime/stack.go`, it will setup the global stack pool, all stacks will be allocated by it.
- `mallocinit`. Defined in `src/runtime/malloc.go`, it will initialize the memory allocator used by Go.
- `cpuinit`. Set up the `internal/cpu` information.
- `alginit`. Defined in `src/runtime/alg.go`, set up the CPU instructions that will be used by some internal algorithms, depending on `cpuinit`.
- `fastrandinit`.
- Initialize the `g0.m` with `mcommoninit`.
- `modulesinit`. Defined in `src/runtime/symtab.go`, initialize the dynamic loaded modules.
- `typelinksinit`. Defined in `src/runtime/type.go`, initialized the dynamic-loaded module type informations.
- `itabsinit`. Defined in `src/runtime/iface.go`, update `itabTable` with the dynamic-loaded modules.
- `stkobjinit`. Defined in `src/runtime/stkframe.go`, this will set up `methodValueCallFrameObjs`, used by later GC module.
- Save the signal mask as `initSigMask`.
- Parse `argv`. Defined in `src/runtime/runtime1.go`.
- Parse environment variables.
- `parsedebugvars`. Defined in `src/runtime/runtime1.go`.
- `gcinit`.
- Set up the number of processors, and trim it accordingly with `procresize`:
  - Grow `allp` as necessary.
  - Initialize all the `p`'s of `allp`.
  - Release old `p`'s.
  - Track the idle and runnable `p`, and return the runnable `p`'s.
- There should be zero runnable `p`, since this is only a bootstrap process.
- Start the world.

### `runtime.mstart`

`mstart0` set up the stack info, then calls `runtime.mstart1` to allocate the m:

- Check whether the current `g` is `m`'s scheduling `g0`.
- Set up the `m.g0`, mostly the same as `newproc`.
- Init the assembly part with `asminit`, ISA-dependent.
- Init the thread-level information with `minit`, OS-dependent.
- If the current `m` is `m0`, spin up an extra `m`, and initialize signals.
- Call `m.mstartfn`.
- If it's not `m0`, acquire the `m`'s p.
- `schedule()`.

## Start a New Goroutine: `runtime.newproc`

In Go, you start a goroutine with the `go` keyword, this compiler will expand this keyword as `runtime.newproc`.

- `getg` to fetch the curreng `g` information.
- `getcallerpc` to fetch the pc register of the caller. This function is expanded to assembly code by the compiler as `ssa.OpGetCallerPC`.
- Run the goroutine from `systemstack`. `systemstack` is used to run the function with a goroutine stack.
- `acquirem` to get the `m` of current `g`.
- Get a free `g` from `p`.
  - First check the `p.gFree`, if empty, try to get from the global `sched.gFree`.
  - Pop a free `g` from `p.gFree`.
  - Clear the `g`'s old stack and allocate a new stack for it.
- If there's no free `g` allocated before, allocate a new one.
  - Create a new `g`.
  - Allocate the memory for system stack size + desired stack size.
  - Change the `g` status to `_Gdead`.
  - Add the new `g` to the global `allgs` list.
- Reserve some extra memory at the top of the stack, and shift `sp` register accordingly.
- Set up the `g.sched`. It contains `sp`, `pc`, and the new `g`'s address.
- Set up the `g.sched`, and get ready for the actual function call.
- Save the caller's pc into `g.gopc`.
- Save the ancestor info into `g.ancestors`.
- Track the profiling information for system and user-defined goroutines separately.
- Switch `g` status to `_Grunnable`.
- Track the stack at GC.
- Generate `g.goid`.
- Put the `g` on the runnable queue.
- Try to add more `p` to execute `g`.

### `runtime.wakep`

`wakep` is called at the last step of `newproc`, tring to run the `_Grunnable` `g`. This step will make sure that there's always a spinning `m` ready to run the new `g`.

- If there's already a spinning `p`, return immediately.
- Lock the current `m`.
- Try to get a spinning `p`.
- Start a new `m` to hold the `p`.

## Scheduling Cycle: `runtime.schedule`

Callers: `mstart0`, `gopark`, `goschedguarded/gopreempt_m`, `preemptPark`, `goyield`, `goexit1`, `exitsyscall`.

Let's try to analyze these callers one by one.

### `runtime.mstart0`

See above.

### `runtime.goschedImpl`

It's a common function to schedule a goroutine:

- Check `g` status is `_Grunning`.
- Change `g` status to `_Grunnable`.
- Detach `g` from the current `m`.
- Put `g` onto the global run queue.
- `schedule()`.

### `runtime.gopark`

See the [last post](https://kassiansun.github.io/posts/go-source-code-part-8/).

### `runtime.gopreempt_m`, `runtime.goschedguarded`

These are wrappers around `goschedImpl`.

### `runtime.preemptPark`

This function is similar to `goschedImpl`, but switches the `g` status to `_Gpreempted`.

### `runtime.goyield`

This function is similar to `goschedImpl`, but without the `_Grunning` check, because it's called from the current goroutine to yield the current `m`.

### `runtime.goexit1`

`goexit1` will finish the current goroutine.

- Change the status to `_Gdead`.
- Release all `g`'s resources.
- Detach `g` from the current `m`.
- Put `g` on the free list.
- If the `g` is locked, kill the thread directly.
- `schedule()`.

### `runtime.exitsyscall`

`exitsyscall` will only be called by syscalls, after the syscall is finished, it will call this function to switch back to the goroutine.

- Try to acquire the old `p` or get one idle `p`. If acquired, no need to do anything, just return immediately.
  - If the old `p` is still `_Psyscall`, switch to `_Pidle` and attach `g` to this `p`.
  - Get a `p` from the global idle `p` list, and attach `g` to this `p`.
  - Switch the `g` status to `_Grunning`.
- Update `g` status to `_Grunnable`.
- Detach `g` from the `m`.
- If `g` can be scheduled, try to get an idle `p`.
- If there's no idle `p`:
  - Put `g` on the global runnable queue.
  - If it's locked, `startlockedm`:
    - Stop the locked `m` of the current `g` if necessary. The current executing `g` on that `m` will be handed off to others.
    - Execute `g`.
  - Else:
    - Stop the m and wait for an available `p`.
    - `schedule()`.
- Else:
  - Notify `sysmon` if necessary.
  - Attach `g` to the `p` and run it.
  - Execute `g`.

### The implementation of `runtime.schedule`

- If the `g` is locked, stop that `m` and run `g` on that `m`. No scheduling in this part.
- CGO is always handled in `g0`, so it should never call the `schedule()`.
- Find a runnable `g` with `findRunnable`
  - Wait for GC stop-the-world.
  - Get the timer information from the current `p`.
  - Try to schedule a trace reader.
  - Try to schedule a GC worker.
  - Get one `g` from the global runnable queue. Run it regularly but not on every cycle to ensure fairness.
  - Mark GC finblock's as ready.
  - If there's a `cgo_yield`, run it.
  - Get one `g` from the local runnable queue.
  - Get one `g` from the global runnable queue.
  - Poll the network.
  - Steal the work from other `p`'s.
  - There's nothing to do:
    - Run GC marking worker.
    - Put back `p` to the global idle list.
    - Poll the network.
    - Stop the current `m`.
    - Try it again until find a `g`.
- Reset the spinning `m`.
- Wake up a `p` for internal goroutines (GC workers, etc).
- If the `g` is locked to an `m`, run `startlockedm`.
- Execute `g`.

## What's Next

- Go's memory allocator.
- Go's stack allocator.
