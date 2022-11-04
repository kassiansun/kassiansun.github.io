---
title: 'Go Runtime Implementations: Select'
date: 2022-11-04T14:14:30+08:00
tags:
  - go
---

- Code Path: `src/runtime/select.go`

## IR stage

IR will walk a `select` statement and generate the code accordingly:

- If there's no `case`, replace the select statement with `runtime.block`, the current goroutine will be blocked forever.
- If there's only one `case`, extract the send or receive operation from the `case` statement.
- Otherwise, convert `case` values to addresses.
- If there's only one `case` with one `default`, replace it with non-block calls `selectnbsend` or `selectnbrecv`.
- Generate a select statement with the list of `send` and `recv` cases, and run `selectgo` on it.
- Run the `case` or `default` based on the returned index of `selectgo`.

## `selectgo`

`selectgo` takes two major arguments: a list of `scase` and a list of the orders of `recv/send`.
It returns the pos of `case` to execute.

- Build a permuted polling order of cases.
- Build a sorted order of locking (sorted by channel address).
- Lock all channels by the order of locking.
- Iterate through all case channels, and find the goroutines on `waitq` or `recvq`. If not empty, send/recv immediately.
  - Closed channels or buffered channels with available slots are also handled during the first round of iteration.
- If there's a default statement, return immediately with no `case` to run.
- Iterate through all case channels, and put the current `g` on the `recvq` or `waitq` of each channel.
- `gopart` the current `g`.
- Once the current `g` wakes up, lock all channels, and dequeue from all the channels except the one we're waking up on.
  - The `recv` status is returned by `sudog.success`
- Return the index and `recv` status.
