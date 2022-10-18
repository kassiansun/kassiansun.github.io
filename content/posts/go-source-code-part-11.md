---
title: 'Go Runtime Implementations: Garbage Collection'
date: 2022-10-18T14:17:39+08:00
tags:
  - go
---

- [Ref](https://go.dev/doc/gc-guide)

## Garbage Collector

- Code Path: `src/runtime/mgc.go`

### `gcinit`

`gcinit` runs after almost everything set up in `schedinit`.

- Set `sweepDrainedMask`
- Initialize `gcController` with `GOGC` (for GC percentage control) and `GOMEMLIMIT` (for memory limits).
- Initialize `work` semaphores.

### `gcenable`

`gcenable` happens in the `main` goroutine, right after `runtime_inittask`.

- Start `bgsweep`
- Start `bgscavenge`

### `GC`

`GC` runs a full garbage collection. Each run will finish the current GC cycle: sweep termination, mark, mark termination, and sweep.
Then it will start a new GC cycle. Note that GC cycles can move forward to more than `N+1`.

- Wait for the current mark phase to finish.
- `gcStart`. `gcStart` can be controlled by different kinds of triggers: heap size, GC period, and GC cycle number.
  - Check whether m is preemptible or running on a non-`g0`.
  - Check the trigger and run `sweepone` until there's nothing to sweep.
  - `gcStart` supports three modes controlled by `debug.gcstoptheworld`: concurrent GC and sweep, STW GC and concurrent sweep, STW GC and STW sweep.
  - Check that all `p.mcache` has been flushed.
  - Start the background marker goroutines.
  - Set up `work` information.
  - Stop the world. This is controlled by `stopTheWorldWithSema` of `proc.go`, it will try to stop all `p` and `g`.
  - Sweep all spans.
  - Clear all `sync.Pool`.
  - `gcController.startCycle`.
  - `gcDrain` -> `gcSweep`.
- Wait for `gcStart` to finish marking and sweeping.
- Check the cycle and run `sweepone` until there's nothing to sweep.
- Wait for the sweeping done.

## Marking

- Code Path: `src/runtime/mgcmark.go`

### Object Scanning

- Code Path:
  - `src/runtime/mbitmap.go`
  - `src/runtime/mcheckmark.go`

GC will scan the module's `.data` and `.bss` section to find all objects. An object can be found by `findObject`, and its GC `markBits`
can be obtained from `mspan`. Large objects will be splitted into "oblets".

### Stack Scanning

- Code Path: `src/runtime/mgcstack.go`

### `gcAssistAlloc`

Goroutines will call `gcAssistAlloc` to assist the GC.

## Sweeping

- Code Path: `src/runtime/mgcsweep.go`

### `sweepone`

## Pacing

- Code Path: `src/runtime/mgcpacer.go`

`gcController` controls how the GC cycles will run.

### `startCycle`

## Scavenge

- Code Path: `src/runtime/mgcscavenge.go`

## What's Next

- Write barriers.
