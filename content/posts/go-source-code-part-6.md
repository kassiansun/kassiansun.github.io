---
title: 'Travesal Through The Go Compiler, Part 6'
date: 2022-09-20T10:39:15+08:00
tags:
  - go
---

## `ssagen.Compile`

### Build SSA for the `ir.Func` (`buildssa`)

- Check `ssaDump` and set the dump flag.
- Initialize `ssagen.state`.
- Push the line number or the parent's line number (if it's missing) to the stack.
- Set up `ssagen.state` flags from `ir.Func` information.
- Set up an empty `ssagen.ssafn` with `ir.Func` and ABI information.
- Allocate the starting block for the current `ssa.Func`.
- Check open-coded defers. <- What's this?
- Do `ABIAnalyze`, get the `abi.ABIParamResultInfo` of function in/out parameters.
- Generate the addresses of the local variables saved in `ir.Func.Dcl`.
- Generate the `AuxCall` for the current function.
- Generate the addresses of the input parameters saved in `ir.Func.Dcl`.
- Generate the addresses of the closure variables saved in `ir.Func.Dcl`.
- Covert to SSA IR. I don't have any experience with SSA, so it's really difficult to understand this part of the code.
  - Insert Phi values. <- What's this?
- Call `ssa.Compile`. It defines an array of `ssa.pass`, and run each of them against the `ssa.Func`.

### Generate SSA (`genssa`)

This step returns a `objw.Progs` to save the machine-level instructions of the function. `objw` saves platform-independent structures of the binary.

- Scan all blocks and generate the code, then call functions of `ArchInfo` to generate the end of the block.
- Check inline marks and mark the functions as inline.
- Resolve branches and jump table.

### Assemble to machine code (`pp.Flush`)

## What's Next

- Read \<\<Compilers: Principles, Techniques, and Tools\>\>
- BuildID
- ABI architecture.
- `types` system.
- `ir` and `ssa` structures.
- escape analysis.
- `objw` structures.
