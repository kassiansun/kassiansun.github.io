---
title: 'Traversal Through The Go Compiler, Part 1'
date: 2022-09-15T08:48:31+08:00
tags:
  - go
---

## Set Up the Environment

`vim-go` doesn't support travesal through the `go` source code, so I switch to `vscode` and it works out of box (with Go plugin && `gopls` installed).

## Start Point (`go run`)

- File: `src/cmd/go/main.go`
- Declaration: `run.CmdRun`
- Function: `run.runRun`
- Key data structures:
  - `build.Context`
  - `work.Builder`
  - `load.Package`
  - `work.Action`

The `runRun` function will go through several stages:

- Check `shouldUseOutsideModuleMode`. This procedure is used by `go run cmd@version`
- `work.BuildInit`. This will setup the build context:
  - Initialize `modload` module. It will check go module flags and set up the flags, no actual module download.
  - `instrumentInit` and `buildModeInit`, and update the default `build.Context` accordingly.
- Get a `work.Builder`. `work.NewBuilder` will check the environment and make sure it's ready to do the actual build.
- Inits a `load.Package` from all `*.go` files passed to `go run`.
  - `load.Package` has a public struct for definitions, and an internal struct for running state.
- Setup `builder.LinkAction`. `LinkAction` will call `cacheAction` (for looking up action cache, not build cache), `CompileAction`, and `installAction` (not for `go run`).
  - `work.Action` is a DAG, the action cache depends on `mode` and package info.
- Initialize a `work.Action`, use the link action initialized at the above stage as dependencies.
- Build the `work.Action` with builder.

## Build Stage (`Builder.Do`)

- Set up the cache trim. There's a `trim.txt` inside `go-build` cache folder, it's used to track the timestamp of last trim action.
- Build the action list, visit the DAG as "depth-first post-order travelsal". The priority will set by the list order, which means deepest will run first.
- `writeActionGraph`. Internal feature, this will dump the DAG as JSON.
- Set up triggers. Triggers are the inverse of dependencies, which means when the dependency ready, it will trigger its root instead of its dependencies.
- The `handle` function will do the actual jobs.
  - Actions are run in parallel, the actual job is defined by `action.Func`.
  - After actions done, update the global state. There's a lock to make sure there's no data races.
  - If all dependencies finished, push the action node to ready queue and signal the `readySema`. Ready queue are protected with the same global state lock.
- Run all actions from the DAG in parallel.

## What's Next

Travesal through the `action.Func` definitions:

- `Builder.build`
- `Builder.link`
