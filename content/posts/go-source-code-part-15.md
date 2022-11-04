---
title: 'Go Runtime Implementations: Slices'
date: 2022-11-04T09:21:06+08:00
tags:
  - go
---

- Code Path: `src/runtime/slice.go`

## `makeslicecopy`

`makeslicecopy` is used for IR patterns like `m = OMAKESLICE([]T, x); OCOPY(m, s)`, IR will rewrite
this specific order of code path and replace it with `OMAKESLICECOPY`. If the elements has no pointer,
SSA will generate code to do a `mallocgc` and `memmove`. Otherwise, the code will be expanded to `makeslicecopy`:

- Check the length of the slice to copy to.
- Do `mallocgc`
- Do `memmove`

## `makeslice` and `makeslice641

`makeslice` is used for `make(slice, len, cap)` statements, if `cap` is missing, by default it will be the same as `len`.

- Try to allocate the memory of `cap` length.
- If `len` > `cap`, allocate the memory of `len` length.
- If `len` < 0 or the memory is too large, panic.

## `growslice`

`growslice` is used for two appending cases: `append(sliceA, sliceB...)` and `append(sliceA, make(slice, len))`.
Note that for the first case you only need `memmove` if `len(sliceA) + len(sliceB) < cap(sliceA)`,
and for the second case you only need to expand the `len` of slice if `len(sliceA) + len < cap(sliceA)`.

- If the element size is zero, return `zerobase`
- Try to double the old `cap`, if it's still less than the new `len`, use the new `len` as the expected size.
  - If the old `cap` > 256, allocate 1.25x old `cap` until it's larget than the new `len`.
- Do `mallocgc`.
- Do `memmove`.

## `slicecopy`

`slicecopy` is used in two cases: after `growslice` to copy the data; used individually as `copy(A, B)`.
Note that `slicecopy` only guarantees `memmove`, it doesn't try to allocate the memory.
