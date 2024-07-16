---
title: "Go Runtime Implementation: CGO"
date: 2024-07-16T13:41:49+08:00
---

- Code Path: `src/runtime/cgocall.go`

The key part here is to switch the stack between `Go` and `CGO`

## Go -> C

### cgocall

- `entersyscall`: this will ensure that CGO codes will not block the Go runtime.
- mark `m` as `incgo`
- `asmcgocall`
- KeepAlive CGO-related `fn`, `arg`, and `mp`

### asmcgocall

- Set-up the stack.

## C -> Go

### cgocallback

- Set-up the stack for `cgocallbackg`, after `cgocallbackg`, the `sp` will point to the previous code
  called `cgocallback`

### cgocallbackg

- callbackUpdateSystemStack
- lockOSThread to pin the `g` on the current `m`
- `exitsyscall`
- `cgocallbackg1`
- undo the previous steps
- `reentersyscall`
