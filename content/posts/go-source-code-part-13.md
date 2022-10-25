---
title: 'Go Runtime Implementations: Typing System'
date: 2022-10-25T17:59:51+08:00
tags:
  - go
---

## Compiling

- Code Path: `src/cmd/compile/internal/ir/expr.go`

During the IR stage, each expression will get type info, it's defined as `miniExpr` and can be assigned a `types.Type` value.

## Static Typing: `types2`

- Code Path: `src/cmd/compile/internal/types2`

### `types` and `types2

- Conversion Code Path: `src/cmd/compile/internal/noder/types.go`

Currently, both packages are in use, but much logic is being migrated to `types2` package.
`types2` package was introduced as part of `go` generic features and has a better code structure than the old `types` package.

### `typecheck`

The `typecheck` package fully depends on the `types` package.

### Type Definitions

- `basic.go`: Predeclared types, boolean, int, string, etc.
- `array.go`
- `chan.go`
- `interface.go`
- `map.go`
- `named.go` -> For any defined types, contains the methods information
- `package.go`
- `pointer.go`
- `selection.go` -> for expressions like `x.f`.
- `signature.go` -> for functions and methods, contains the receiver information.
- `slice.go`
- `struct.go`
- `tuple.go` -> for multiple variable assignments, not a real type.
- `typeterm.go` -> for describing elementary type sets, used by `union.go`
- `union.go` -> a list of terms.

#### Scopes

A scope contains a list of children, objects, and the reference to its parent scope.

##### `universe.go`

`universe.go` defines the root `Scope`.

## Runtime Implementations

### IR Walking

- Code Path: `src/cmd/compile/internal/walk/walk.go`

If we want to `make` a new slice, the compiler will replace the IR tree with the corresponding runtime implementations.
Generally, the replacement happens at the walking logic, the entry function is `Walk(fn *ir.Func)`.

### Function Declarations

- Code Path: `src/cmd/compile/internal/typecheck/builtin.go`

The `typecheck` package declares all builtin functions, they're replaced dynamically through two ways:

- `typecheck.LookupRuntimeFunc`
- Dynamically emit by compiler through stubs defined in `./cmd/compile/internal/typecheck/_builtin/runtime.go`
