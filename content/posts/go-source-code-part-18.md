---
title: 'Go Runtime Implementations: Timer Scheduling'
date: 2022-11-07T08:49:37+08:00
tags:
  - go
---

- Code Path: `src/runtime/time.go`

## `time.startTimer` (implemented as `addtimer`)

Each `p` has a `timers` list. When a user is adding a new timer, it will first try to clean up the `timers` list,
then adds the new timer. Each `timers` list is a heap sorted by `timer.when`, and will be updated during add/delete timers.

### `cleantimers`

`cleantimers` only checks the head of the `timers` list.

- Check if it's `timerDeleted`, delete it from the `timers` list, and update `p`'s `timer0When`.
- Check if it's `timerModifiedEarlier` or `timerModifiedLater`, delete it from the `timers` list and add it back to put it at the right position in the list.

### `doaddtimer`

`doaddtimer` link the time to the `p`'s `timers` list, and sift up the heap to put it in the right position.

## `time.stopTimer` (implemented as `deltimer`)

`deltimer` doesn't delete a timer, the code calling `deltimer` might be running on a different `p` from the timer's `p`.
It will update the timer's status to `timerDeleted` and wait for `p` to clean it up.

### `dodeltimer`

`dodeltimer` swap the timer at location `i` with the timer at the last index, and sift down the heap at i-position.

## `timer.modTimer` (implemented as `modtimer`)

`modtimer` updates the when and period of a timer, and returns whether it was modified before it was run.

- Try to update the timer status to `timerModifying`. If the timer is being modified, wait for it to complete.
- If the timer was deleted, add the timer back to the current `p`'s list.
  - Switch the timer to `timerWaiting`.
  - Call `wakeNetPoller` to determine whether we should serve the timer.
- Else, get the `p` of the timer and only update the `nextwhen` information, so the correct `p` will try to pick it up and update it accordingly.

### `resettimer`

`resettimer` will reset the timer to fire at a new `when` value.

### `moveTimers`

`moveTimer` happens when a `p` is destroyed, it will add these timers one by one to the new `p`'s timers list.

## Scheduling of timers

Timers will be picked-up during `findRunnable`.

### `checkTimers`

- Check `timer0When` and whether we should continue to clean up timers. All calculations are based on fields updated during add/mod/del timers.
- Do `adjustTimers`
- Pick up one timer from the list by `runtimer`, run it, or return the next timer's `when` value as `pollNext`.

### `adjustTimers`

- Delete the timers with `timerDeleted` status.
- Delete the timers that have been modified, and add them back later.
- If it's being modified, wait for it to complete.

### `runtimer`

`runtimer` will pick up the head of `timers` list if it's `timerWaiting`, and update its status to `timerRunning`.
The actual running is done by `runOneTimer`:

- If it has a period value, calculate the new when and sift down the timers list.
- Else delete it from the timers list.
- Release the timers lock -> Run the timer function -> Acquire the timers lock.
