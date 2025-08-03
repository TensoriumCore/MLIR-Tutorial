Creating an MLIR dialect out-of-tree means describing your operations in TableGen (.td) files and then implementing the connection in C++. Here’s how and why each part matters—presented as a logical workflow, not as a step-by-step tutorial for beginners, but as a technical, narrative explanation.

---

The process begins by defining the dialect itself in a file like `include/TutoDialect/TutoDialect.td`. This file is concise: it gives MLIR the dialect’s name (`tuto`) and its C++ namespace, which enables TableGen to generate all C++ symbols and the syntax used in MLIR files. For example:

```tablegen
include "mlir/IR/OpBase.td"

def Tuto_Dialect : Dialect {
    let name = "tuto";
    let summary = "A Tuto out-of-tree MLIR dialect.";
    let description = [{
        This dialect is an example of an out-of-tree MLIR dialect designed to
        illustrate the basic setup required to develop MLIR-based tools without
        working inside of the LLVM source tree.
    }];
    let cppNamespace = "::mlir::tuto";
}
```

On the same principle, you then describe the dialect’s operations in a dedicated TableGen file, typically `include/TutoDialect/TutoDialectOps.td`. This file gathers all operations: each operation is declared with its input/output types and names, and its MLIR assembly format. For instance, an addition of floats:

```tablegen
include "TutoDialect.td"
include "mlir/IR/AttrTypeBase.td"
include "mlir/IR/DialectBase.td"
include "mlir/Interfaces/InferTypeOpInterface.td"
include "mlir/Interfaces/SideEffectInterfaces.td"


def AddOp : Tuto_Op<"add", [Pure]> {
  let summary = "Addition";
  let description = [{
    Addition operation between two values.
  }];

  let arguments = (ins F64:$lhs, F64:$rhs);
  let results = (outs F64:$res);

  let assemblyFormat = [{
    $lhs `,` $rhs `:` type($lhs) attr-dict
  }];
}
```

This file is the blueprint for generating the full C++ class for the operation via TableGen, ensuring parsing, syntax, and printing are consistent and correct.

Once these definitions are written, TableGen (invoked by CMake during the build) generates all the backend C++ (headers and intermediate sources). Then, you just need to implement the minimal glue in C++: the main dialect file, for example `lib/TutoDialect/TutoDialect.cpp`, is responsible for registering all operations of the dialect within MLIR. This is done with a simple `initialize()` method that adds your operations to the dialect’s table. Nothing magic—this is the key that makes your operations usable in tools like `mlir-opt` or your own binary (`tuto-opt`).

Note: the C++ operations file (`TutoDialectOps.cpp`) is usually empty at first, unless you want to add custom verification, builders, or canonicalization logic. The parsing, syntax, and type signatures are already handled by TableGen.

With these files in place, you simply build the project. The dialect can then be used in a `.mlir` file like:

```mlir
func.func @add_example(%arg0: f32, %arg1: f32) -> f32 {
  %res = tuto.add %arg0, %arg1 : f32
  return %res : f32
}
```

And you can test it using your binary:

```bash
./bin/Tuto-opt test/TutoTest.mlir
```

Result: your dialect and operations are fully integrated into MLIR and ready to be extended—add types, patterns, lowerings, whatever you need.

---

**Logical summary:**

- `.td`: all declarative stuff—syntax, signatures, metadata.
    
- TableGen: generates classes/headers.
    
- `.cpp`: registers and connects things, and (optionally) advanced logic.
    
- `.mlir`: lets you write and test your operations at a high level.
    
- `tuto-opt`: your binary for parsing, validating, and manipulating your dialect.