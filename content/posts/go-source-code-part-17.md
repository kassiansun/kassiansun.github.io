---
title: 'Go Runtime Implementations: Network Polling'
date: 2022-11-05T08:56:11+08:00
tags:
  - go
---

- Code Path: `src/runtime/netpoll.go`

Network polling is platform-dependent, the following is of BSD family operating systems (based on [kqueue](https://www.freebsd.org/cgi/man.cgi?query=kqueue&apropos=0&sektion=2&manpath=FreeBSD+14.0-current&arch=default&format=html)).

## `netpollinit`

- Initialize a `kqueue` descriptor, and set `CLOSEXEC` flag on it.
- Initialize a non-blocking pipe, and use the reader/writer for `netpollBreak`.
  - `netpollBreak` will try to write to the pipe and interrupt the `kevent`.

## `netpoll`

`netpoll` is used to check the connections and return a list of `g` ready to run. It can block for `delay` ns at most.
The `delay` parameter will be passed to `kevent` syscall.

- Call `kevent`, fetch at most 64 `keventt`.
- Iterate through each `keventt`.
  - If it's a blocking call and there's some data on `netpollBreakRd`, read 16-byte data from it.
  - Get the `mode` value based on `keventt.filter`.
  - Get the `pollDesc` information from `keventt.udata`.
  - Call `netpollready` to populate the `gList`.
- Return the `gList`.

### `netpollready`

`netpollready` calls `netpollunlock` to swap the pending `g` of `pollDesc` to `pdReady`, and return the pending `g`.

### `netpollblock`

- Tries to swap the pending `g` of `pollDesc` to `pdWaiting`, or return immediately if it's `pdReady`.
- `gopark`.
- Check the pending `g` and return its status.

## `internal/poll.FD`

`poll.FD` is the underlying implementation of file descriptor used by `net` package.

### Open

- If the `FD` is not pollable, there's nothing to do.
- Initialize a `pollDesc` and use it as `runtimeCtx`. This is done by `poll_runtime_pollOpen`:
  - Get a `pollDesc` from cache.
  - Lock the `pollDesc, set the file descriptor and initialize all other fields.
  - Register read and write event with `kevent`.

### Close

- Call `increfAndClose`, wake up all waiters and notify the closing status.
- Call `poll_runtime_pollUnblock`, wait for the g ready to consume the `pollDesc`, and schedule these `g` to run later.
- Release the `FD`, and return the underlying `pollDesc` back to the cache, then close the system's file descriptor.
- Wait for the close semaphore.

### Read

- Acquire the read lock.
- Reset the descriptor mode to `r`.
- Do syscall to read the file descriptor, block and wait if necessary.

### Write

- Acquire the write lock.
- Reset the descriptor mode to `w`.
- Do syscall to write the file descriptor, block and wait if necessary.
