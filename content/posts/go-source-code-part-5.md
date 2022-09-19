---
title: 'Travesal Through The Go Compiler, Part 5'
date: 2022-09-19T10:54:34+08:00
tags:
  - go
---

## `irgen.generate`

- Check each file's pragma list and `DeclList`.
- Generate `ir.Decl` for `type` declarations.
- Generate `ir.Decl` for other declarations.
- Process later functions, this step is for the same purpose as type check.
- Type-check `CallExpr` again.
- Check missing function bodies.
- Build generic instantiations. It scans calls and generated needed methods.
- Remove all generic `Decl`'s.

After `irgen.generate`, `g.target.Decls` will be the final `Decl`'s to generate.

## `enqueueFunc` and `compileFunctions`

This step is the compile step of `cmd/compile`, it happens after type checking and IR generation.

### `enqueueFunc`

There're some operations unsafe for parallel, `enqueueFunc` will handle these steps single-threaded.

For body-less functions:

- Initialize LSym and with-body text symbol (if it has a function body).
- Calculate the size and alignment of the functions.
- Generate the ABIConfig for body-less functions (`ssaConfig.ABI0`).
- Analyze the function call parameters, this step depends on the ABI's number of registers. Each parameter will be passed by the stack or registers.
- Put the LSym of function to the global Link context.
- Generate the `ArgInfo` for body-less functions, and write the generated LSym.

For other functions, call `prepareFunc` to set-up the `ir.Func`:

- Initialize LSym and with-body text symbol (if it has a function body).
- Calculate the size and alignment of the functions.
- Walk all statements of the function and generate the `ir` structures. Different kinds of statements will have different walking logic.

### `compileFunctions`

- If race detection is enabled, compile the code in random order. Otherwise, compile the largest function first.
- For each function:
  - Call `ssagen.Compile` for each `ir.Func`.
  - Recursively compile the closures inside the function.

## Whats'Next

- ssagen.Compile
