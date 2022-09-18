---
title: 'Travesal Through The Go Compiler, Part 4'
date: 2022-09-18T14:09:58+08:00
tags:
  - go
---

## `noder.LoadPackage`

### Parsing

`noder.LoadPackage` will call `syntax.Parse` to generate the syntax tree of all package files.

- Initialize a `compile/internal/syntax.scanner`
- Advance `scanner` to the next token.
- Start parsing.
  - Check the `package` declaration first, and take the package name from the parsed pragma.
  - Note that the parser is expecting a `;` token, but this is automatically generated at the scanner, so users don't have to write the `;`. And parsing errors will not always abort the parser loop.
- Parse top-level declarations: `const` `type` `var` `func`
  - Each type has its function to parse the whole syntax tree.

### Type Checking

After `syntax.Parse`, `check2` is called to do the IR generation.

- Merge `posMap` from all files (`noders`). posMap contains the mapping from `syntax.PosBase` to `src.PostBase`
- Build type checking context and configuration, and do the type checking (`types2.Config.Check`). It will Call `NewChecker.Files` to check all `syntax.File` extracted from `noders`.
  - Check and initialize `check.files`
  - Check each file's `DeclList`
    - Check and import the `import`, build the import map and declare the objects.
    - Declare `const` `var` `type` `func` objects.
  - Check object names.
  - Associate methods with base types.
- Package objects.
  - Sort all objects by in-source order.
  - Collect methods of their base types.
  - Declare the objects in the following order: non-alias type, alias type, and others. The objects are annotated with three different states (copied from comments):
    - an object whose type is not yet known is painted white (initial color)
    - an object whose type is in the process of being inferred is painted grey
    - an object whose type is fully inferred is painted black
- Processed delayed actions. There're some types that can't be fully checked at the first round of checking, so we need to delay these actions and check them later.
  - The delayed actions are marked by `check.later`, you can check the callers of this function to view all delayed actions.
- Clean up the types. It's marked by `check.needsCleanup`.
- Compute the init order of package variables. It builds the dependency graph, then sorts them by the number of incoming dependencies and source order.
  - It's a topology sort to get an ordered list of the graph nodes, dependency cycle is also checked here.
- Check unused imports.
- Records untyped values. This is tracked by `check.rememberUntyped`.
- Run `check.monomorph`. This step is to detect unbounded instantiation cycles.

## What's Next

- IR Generation
