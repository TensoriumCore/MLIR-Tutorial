
MLIR (*Multi-Level Intermediate Representation*) is a modular compiler infrastructure developed within the LLVM project.  
It allows representing and transforming programs at multiple abstraction levels, using an extensible architecture made of *dialects*.

In many projects, it is necessary to define custom operations, types, and transformation rules. This is done by creating a **custom MLIR dialect**.

---

## Why an *out-of-tree* dialect?

There are two main ways to develop an MLIR dialect:
- **In-tree**: integrated directly into the LLVM repository (like `arith`, `math`, or `linalg` dialects).
- **Out-of-tree**: developed in a separate repository, compiled against an existing LLVM/MLIR installation.

The *out-of-tree* approach is the most flexible for independent projects.  
It allows:
- Maintaining a repository separate from LLVM.
- Evolving quickly without being tied to upstream LLVM commits.
- Testing against different LLVM versions (MLIR changes quickly).

---

## Tutorial goals

This tutorial will guide you step by step through:
1. Setting up an MLIR build environment.
2. Creating a minimal *out-of-tree* dialect (with operations and types).
3. Writing a simple pass.
4. Lowering to standard dialects (`arith`, `math`).
5. Testing the dialect with `lit` and `mlir-opt`.

> This tutorial is up to date for **LLVM 22**.

- [Next section: CMake Environment](Environment-Setup.md)