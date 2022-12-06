---
title: 'Redis Internal: Hash Types'
date: 2022-11-26T10:15:35+08:00
tags:
  - redis
---

## The Generic Hash Implementation

- Code Path: `src/t_hash.c`

The generic hash type is a wrapper around `listpack` and `dict`, it will switch between two different
implementation and call the routines accordingly.

## Listpack Implementation

- Code Path: `src/listpack.c`

Listpack is a densed storage of a series of keys and values, it supports storing both integers and strings.
If the hash size is lower than `hash_max_listpack_value`, it will be saved as a `listpack`.
If the hash size is higher than `hash_max_listpack_value`, it will create a `dict` and copy
the data from `listpack` to the `dict`.

## Dict Implementation

- Code Path: `src/dict.c`

Redis' `dict` is a pretty-standard hash map implementation, with two hash table swapping on rehashing.
The redis' rehashing implementation is incremental, similar with Go's `map` implementation.
