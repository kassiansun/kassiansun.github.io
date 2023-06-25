---
title: "Go Runtime Implementations: Defer, Panic and Recover"
date: 2023-06-25T14:18:47+08:00
---

- Code Path: `src/runtime/panic.go`
- Ref: [Defer, Panic, and Recover](https://go.dev/blog/defer-panic-and-recover)

# Defer

## Two methods of `defer`

Copied from the comment:

> The older way involves creating a
> defer record at the time that a defer statement is executing and adding it to a
> defer chain. This chain is inspected by the deferreturn call at all function
> exits in order to run the appropriate defer calls.

> A cheaper way (which we call open-coded defers) is used for functions in which no defer statements occur in
> loops. In that case, we simply store the defer function/arg information into
> specific stack slots at the point of each defer statement, as well as setting a
> bit in a bitmask. At each function exit, we add inline code to directly make
> the appropriate defer calls based on the bitmask and fn/arg information stored
> on the stack. During panic/Goexit processing, the appropriate defer calls are
> made using extra funcdata info that indicates the exact stack slots that
> contain the bitmask and defer fn/args.

## Defer implementations

There're three ways of defer:

- deferproc
- deferprocStack
- inline defer

`deferproc` and `deferprocStack` will update the list `gp._defer`.

# Goexit

`Goexit` calls all `gp._defer`, and then calls `goexit1()` to release the resources.

# Panic

## Runtime panics `goPanic*`

Copied from the comment:

> Since panic{Index,Slice,shift} are never called directly, and
> since the runtime package should never have an out of bounds slice
> or array reference or negative shift, if we see those functions called from the
> runtime package we turn the panic into a throw. That will dump the
> entire runtime stack for easier debugging.

## `gopanic`

- Checks whether it's panicking on system stack, during mallocing, with preempt off, or holding lock.
- Add itself to the panic list `gp._panic`
- `addOneOpenDeferFrame`
- Handle the loop of `defer` and `panic`
  - The most important thing is that panic overwrites previous panic

## `gorecover`

Copied from the comment:

> Must be in a function running as part of a deferred call during the panic.

> Must be called from the topmost function of the call
