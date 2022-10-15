---
title: 'Go Runtime Implementations: Goroutines'
date: 2022-10-12T14:42:48+08:00
tags:
  - go
---

## Preface

- Code path: `src/runtime/proc.go`
- Data structures are defined under `src/runtime/runtime2.go`.

### Concepts

Copied from source code:

```go
// G - goroutine.
// M - worker thread, or machine.
// P - processor, a resource that is required to execute Go code.
//    M must have an associated P to execute Go code, however it can be
//    blocked or in a syscall w/o an associated P.
```

## `main`

- Set up stack size.
- Fork a new `m` and run the `sysmon` function, except the wasm platform.
- Lock the main `m` and `g`.
- Run the runtime `initTask`'s.
- Enable GC.
- Run the main `initTask`'s.
- Run `main_main`.

### `runtime.sysmon`

`sysmon` is a global worker runs regularly to ensure the system state consistency.

- Check deadlocks.
- Poll the network every 10ms.
- Try to force a GC.

## `runtime.getg`

`runtime.getg` is generated directly at `ir`-level, which means there's no functio definition, but code expansion instead.
This operation is defined as `ir.OGETG` and `ssa.OpGetG`, it's written as a platform-specific implementation. For amd64, the implementation is defined
by `src/cmd/compile/internal/amd64/ssa.go` as `getgFromTLS`, all `getg` calls will be replaced by assembly code directly.
The assembly code just copies the TLS from the ISA-specific register to the target register.

## `runtime.acquireSudog`

`acquireSudog` returns a `sudog`, which is a pointer to g and information on the wait list. `sudog` is generated when you want to put a g on the
wait list and wait for something to happen. There's a local cache of `sudog` for each `p`.

## `runtime.gopark`

- Park the current `g` on m.
- Change the `g` status to `_Gwaiting`
- Drop the `g` as the current `g` on m.
- Runs the `waitunlockf`, if the function returns, execute it immediately, the function won't return in this case.
- Run the scheduler.

## `runtime.goready`

- Update the `g` status to `_Grunnable`
- Put the `g` on the `m.runnext`, if there's an old `m.runnext`, replace the `gp` and put the old `m.runnext` on the runnable queue.
- Schedule an `m` to run the `g`.

## What's Next

- Goroutine scheduler.
- GC implementation.
