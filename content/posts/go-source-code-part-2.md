---
title: 'Travesal Through The Go Compiler, Part 2'
date: 2022-09-16T15:16:23+08:00
tags:
  - go
---

## `Builder.build`

`Builder.build` generate an archive for a single package.

- `Builder.build` gets building information from two sources:
  - `internal/cfg` package. It contains some global variables saving build flags.
  - `load.Package`. It contains information for the current building package.
  - `Builder.build` will build a `need` flag, finish the step and uncheck the finished bits one by one.
- Set up caching information by action, its package, and package's target.
- Set up objects output directory.
- Setup `cgo` cache & `vet` cache.
- Set up building target directory.
- Set up non-Go files overlay. This step is to copy the non-Go files to the object directory.
- Preprocess coverage files and replace the file path of original `*.go` files.
  - It runs `go tool cover -mode=b.coverMode -var="varName" -o dst.go src.go` to generate the coverage files.
- Build `cgo` files.
  - After the build, the `cfiles` will be cleared and generated go files will be added to `gofiles`
- Cache all source files. It's all `*.go` files now
- Generate import configuration and package dependencies information.
- Generate embedded files configuration.
- Run `BuildToolchain.gc`. This is the actual `go` build stage.
- Run `BuildToolchain.cc`. This is the actual `cgo` build stage.
- Run `BuildToolchain.asm`. This is the actual `*.s` build stage.
- Run `BuildToolchain.pack`. This will generate the object archive (`*.a`).
- Update the `BuildID` accordingly.

## `Builder.link`

`Builder.link` links a package and generates the final binary. It's very simple compared with the build stage.

- Generate the link import cfg
- Set up linking target directory.
- Run `BuildToolchain.ld`.
- Update the `BuildID` accordingly.

## What's Next

- BuildToolchain steps
- What's BuildID and how it's generated and used.
