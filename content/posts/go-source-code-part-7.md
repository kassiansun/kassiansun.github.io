---
title: 'Go Runtime Implementations: Channel'
date: 2022-09-29T11:24:26+08:00
tags:
  - go
---

- Path: `src/runtime/chan.go`

## Compilation

When you make a chan with `make` function, the compiler will expand the expression to the `makechan` implementation. The actual expansion happens
at `cmd/compile/internal/walk/expr.go`. The runtime will determine whether to use `makechan` or `makechan64`.

## Type Definitions

### `type _type`

`_type` is used as the internal representation of a `go` type. The same structure is defined multiple times across the `go` runtime.
`_type` stores the following fields:

### `type chantype`

`makechan` only uses `chantype.elem`, the other fields are used by the type system.

### `type waitq`

`waitq` is a queue of waiting goroutines. There's a special flag to check whether goroutine has grabbed the channel locks but not removed itself from `waitq`.
If the flag is true, `dequeue` will continue and try to find the next valid goroutine.

### `type hchan`

`hchan` used a circular queue to save the buffered channel data.

## Runtime Implementation

### `makechan`

- Check the element's size and alignment, and the expected `chan`'s size.
- Allocate the memory for `hchan`. The memory allocation is very dense and the runtime will try to utilize the memory at its best effort.

### `chansend`

- Check whether the channel is unresponsive and return.
- Lock the channel
- Check whether the channel is closed.
- Get a receiver from `waitq`, and send the data directly to that goroutine, then return true.
- If the channel buffer is not full yet, `memmove` the data to the circular queue, then return true.
- If it's not a blocking operation, return false.
- Block the current goroutine, and put it on the `sendq`.

### `send`

`send` will send the data to the target goroutine. It happens when the target goroutine is waiting and the data arrived on the channel.
The `send` operation will copy the data directly to the target goroutine's stack, and make the target goroutine as ready state.

### `chanrecv`

- Check whether the channel is unresponsive and return.
- Lock the channel
- If the queue is closed and the circular queue is empty, return.
- If there's a waiting goroutine on `sendq`, `recv` immediately and return.
- Move the data from the circular queue to the target memory address.
- If it's not a blocking operation, return false.
- Block the current goroutine, and put it on the `waitq`.

### `recv`

`recv` happens under a similar condition to `send`.

- If the channel is unbuffered, `recvDirect` from the sending goroutine to receiving goroutine.
- Read the data from the circular queue first, and move the data to the target memory address.
- Mark the sending goroutine as ready to run. Note that the receiving goroutine is already running.

## What's Next

- Go's typing system
- Go's routines implementation.
