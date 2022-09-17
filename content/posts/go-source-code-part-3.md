---
title: 'Travesal Through The Go Compiler, Part 3'
date: 2022-09-17T09:29:52+08:00
tags:
  - go
---

## BuildToolchain

BuildToolchain is initialized as `noToolchain{}` by default, then it's set dynamically to `gccgo` or `gc`.
Both implementations are wrappers around the binary toolchains installed on the machine. For `gc`, the delegations are as follows:

- `BuildToolchain.gc` - `base.Tool("compile")`
- `BuildToolchain.cc` - `base.Tool("cgo")`. Actually `cgo` has been executed before the compile and `cfiles` has been cleared, so `BuildToolchain.cc` should never be called and it always returns an error.
- `BuildToolchain.asm` - `base.Tool("asm")`
- `BuildToolchain.pack` - `base.Tool("pack")`
- `BuildToolchain.ld` - `base.Tool("link")`

## `cmd/compile` aka base.Tool("compile")

- Entry point: `internal/gc.Main`

It's a standalone binary which means all packages path assumes a relative path to `go/src/cmd/compile`.
`gc.Main` will get an `archInit` function as the parameter, this function populates the `ssagen.ArchInfo` which contains necessary architecture information.

- Initialize a new `obj.Link` as context.
- Parse compiler flags - `internal/base.CmdFlags`.
- Initialize `types.Pkg` information.
- Save DWARF info from compiler flags.
- Initialize `ssagen` by OS environments and initialize the table, also initialize other architecture-specific `types` information.
- Initialize `typecheck`
- `noder.LoadPackage` - This is the main step for compiling and type checking, after this step, `typecheck.Target` (type of `ir.Package`) will be ready to use.
- Initialize `ssagen` configuration.
- Build the `init` function for the package.
- Eliminate dead codes.
- Compute Addrtaken for names. <- What's this?
- Wrapping the types, the imported types are assumed to be wrapped already.
- `devirtualize.Func` all `Decls` with `Op == ir.ODCLFUNC`
- Build init task.
- Generate ABI wrappers.
- Do escape analysis.
- Compile functions in parallel.
- Dump everything to the object file.

## What's Next

- `ssa` architecture.
- ABI architecture.
- `types` system.
- `ir` structure.
- escape analysis.
- Object file structure.
