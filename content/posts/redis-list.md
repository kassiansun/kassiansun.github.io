---
title: 'Redis Internal: List Types'
date: 2023-01-03T11:55:05+08:00
---

## The Generic List Implementation

- Code Path: `src/t_list.c`

It's a Redis command wrapper of `listpack` and `quicklist`.

## Listpack Implementation

The same implementation of [hash type](/posts/redis-hash/).

## Quicklist Implementation

- Code Path: `src/quicklist.c`

If the size of the list exceeds a certain number, Redis will try to convert a `listpack` to `quicklist`.
A `quicklist` is a double-linked list of `listpack`:

- If the data is too large, it will store the data as a "plain" node, without encoding it as `listpack`.
- If the data size exceeds the size limit of `listpack`, it will create a new list node to store it.
  - If it's in the middle of the node, the node will split into two nodes.
- If the list node is empty (after deletion), it will get removed from the list.
- The list node can be compressed, but not the head or the tail.
- The pop operation is a combination of get & delete.
