---
title: 'Go Runtime Implementations: Stack and Memory Management'
date: 2022-10-16T15:48:13+08:00
tags:
  - go
---

## Stack Management

- Code Path: `src/runtime/stack.go`

Stack pools are initialized before `mheap_`, because we don't allocate any memory during this stage.
Right after the stack pools initialized, the `mheap_` will be initialized so later stack allocation is possible.

### `stackpool`

`stackpool` is managed by the "order" (based on the ratio of stack size and `_FixedStack`) of the stack, each order of stack has its own stack pool.
In `stackinit`, the stack pool will be initialized right after `moduledataverify`.

Each stack pool item is a list of `mspan`, recording the start address of the memory zone and the number of pages allocated.

### `stackLarge`

`stackLarge` is similar to `stackpool`, but managed by log2(number of pages).

### `stackalloc`

All `go` stacks will be allocated by `stackalloc`. It will return a new `stack` structure tracking the low and high of the stack address.
`stackalloc` must be called by `g0`.

- Allocate small stacks from the global `stackpool` or `m`'s own stack cache.
- Allocate large stacks from `stackLarge`, or manually by `mheap.allocManual`

### `stackfree`

`stackfree` will not return the memory to the OS, instead, it just puts the memory back to `stackpool` or `stackLarge`.

## Memory Management with `mheap`

- Code Path: `src/runtime/mheap.go`

`mheap_` is an instance of `mheap` struct, which serves as the main malloc heap.

### `mSpanList`

It's not a red-black tree, so querying a free zone is done by iterating through the list with a hint index `freeindex`.
Other fields are for the GC process.

### `mheap.alloc` and `mheap.allocManual`

The difference is `alloc` will try to `mheap.reclaim` the required number of pages, `allocManual` won't.
The underlying allocation strategies are handled by `mheap.allocSpan` and controlled by `spanAllocType`.

### `mheap.allocSpan`

`allocSpan` is the main function to allocate a `mspan`

- Try to get from the small allocation from the page cache first (each `p` has its own page cache). This cache can be accessed without locking.
  - If there's no cache, allocate a new one with `pageAlloc.allocToCache`.
- Try to get a mspan from `p.mspancache`.
- Try to find a free region by `pageAlloc.find`.
  - Calculate the `extraPages` to align the physical page.
  - If not found, grow the heap by `mheap.grow`, and find again.
  - Align up the base address, and mark the region as allocated.
- If it doesn't need physical page alignment, the allocation flow is different:
  - Try `pageAlloc.alloc`.
  - If failed, grow the heap and `pageAlloc.alloc` again.
- Allocate a new `mspan`.
- Initialize the `span` with allocate base address.
- Check whether it needs to zero the allocated `mspan`.
- Initialize the mark and allocation structures if it's a heap allocation. Not for `allocManual`.
- Do the memory scavenge.

### `mheap.grow`

`grow` will allocate a new memory region from the OS, and track the region internally.

## `pageAlloc`

- Code Path: `src/runtime/mpagealloc.go`

`pageAlloc` is the main data structure to manage the pages.

### `pageAlloc.alloc`

`alloc` will first try to find a chunk from the radix tree directly, then fallback to `pageAlloc.find`.

### `pageAlloc.find`

`find` search the radix tree to find a contiguous memory region.

### `pageAlloc.scavenge`

- Code Path: `src/runtime/mgcscavenge.go`

`scavenge` is used to return the physical page back to the OS.

## `pageCache`

- Code Path: `src/runtime/mpagecache.go`

`pageCache` is a per-`p` cache of pages, used for small allocations.

### `pageAlloc.allocToCache`

`allocToCache` gets a new page cache for `p`.

### `pageCache.alloc`
