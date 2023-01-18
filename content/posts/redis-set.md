---
title: 'Redis Internal: List Types'
date: 2023-01-18T14:21:09+08:00
---

## The Generic Set Implementation

- Code Path: `src/t_set.c`

It's a wrapper around the actual set type. The set can be one of the following types:

- An `intset`. Even if the member is a string, Redis will try to convert it to an integer with `string2ll`.
- A `listpack`. Used for small sets.
- A `dict`. Used for large sets.

## Listpack Implementation

The same implementation of [hash type](/posts/redis-hash/).

## Dict Implementation

The same implementation of [hash type](/posts/redis-hash/).

## Intset Implementation

- Code Path: `src/intset.c`

An `intset` is a dense sorted array with multiple levels of encoding for different sizes of integers.

- All integers are sorted, before inserting or deletion, you need a binary search to locate it.
- Before inserting, call `zrealloc` to allocate one more slot.
- Before deletion, find the position of the element and `memmove` the slots following the position.
