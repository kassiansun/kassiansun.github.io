---
title: 'Go Runtime Implementations: Map'
date: 2022-11-23T16:51:25+08:00
tags:
  - go
---

- Code Path: `src/runtime/map.go`

## `makemap`

Each `hmap` has a `hash0` as seed hash. When the compiler generates the typing information for a map, the `hasher`
function will be calculated with `genhash` function (code path: `src/cmd/compile/internal/reflectdata/alg.go`).

If the map is not escaped, the `hmap` will be allocated by the compiler on the stack.

Then `makemap` will allocate an array of `bucket`, the type of `bucket` is generated at compiling stage by `MapBucketType`
(code path: `src/cmd/compile/internal/reflectdata/reflect.go`).

When the map is growing, there will be two different buckets, one is for storing the old data and one is for storing the new data.

## `mapdelete`

`mapdelete` is used to delete a key from the `hmap`. All modifications to a map will mark the `flags` with `hashWriting`.
Before map modifications, we'll try to `evacuate` the elements from the old buckets.

Note that a for-loop to delete all map keys is optimized as a `mapclear` call by the compiler.
An empty map will reset its `hash0` after the last element deleted.

## `mapaccess`

When trying to access a map, `mapaccess` will check the `flags` for concurrent writing, and dynamically switch the destination based
on whether the corresponding bucket has been evacuated. Then `mapaccess` will iterate the bucket to find the key.

## `mapiterinit`

The `hiter` struct is a snapshot of the current status of the map, so even though the map grew later, the iterator can point to the
correct underlying data.

## `mapassign`

`mapassign` will find the bucket for the specified key, and return an address to save the value.

### `hashGrow`

hashGrow happens when the the map is too full. It will grow by either doubling the buckets, or do a same-size growing.
Whether it's a same-size growing will determine whether we should use the current `B` value as an indicator of bucket size or `B - 1`,

The hashing logic doesn't change after a hash growing, but the hask masking will change, so if we don't track the growing strategy,
it's impossible to `evacuate` data from the `oldbuckets` to the `buckets` correctly.

If the buckets are full, there'll be some extra buckets chained behind it to save the overflowed elements.
