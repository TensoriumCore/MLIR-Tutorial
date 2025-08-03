
# Environment Setup

## Installing LLVM and MLIR

To develop an _out-of-tree_ MLIR dialect, you must have LLVM and MLIR installed on your system.

There are two possible approaches:
### 1. Build LLVM/MLIR from source (recommended)

```bash
git clone https://github.com/llvm/llvm-project.git
cd llvm-project

export PREFIX=/opt/local/libexec/llvm  #install dir

mkdir build && cd build
cmake -G Ninja ../llvm \
  -DCMAKE_INSTALL_PREFIX=$PREFIX \
  -DLLVM_ENABLE_PROJECTS="mlir;clang;lld" \
  -DLLVM_TARGETS_TO_BUILD="X86" \     # or any other ISA target
  -DLLVM_ENABLE_RTTI=ON \             # don't forget this flag !!
  -DMLIR_ENABLE_BINDINGS_PYTHON=OFF \ # you can set ON if you have Pybind11
  -DCMAKE_BUILD_TYPE=Release


ninja -j64   #64 cores because of my CPU but set for your own
sudo ninja install
```

> ⚠️ Make sure `mlir-opt --version` returns the expected version.

---

## Project Structure

We will use **CMake** to build the _out-of-tree_ dialect. Basic folder layout:

```
MLIR-Tutorial/
├── CMakeLists.txt
├── include/
│   └── TutoDialect/
│       ├── TutoDialect.h
│       ├── TutoDialectOps.h
|       ├── TutoDialect.td
|		└── TutoDialectOps.td
├── lib/
│   ├── CMakeLists.txt
│   └── TutoDialect.cpp
├── test/
│   └── TutoTest.mlir
```

---

## Minimal CMake

Root `CMakeLists.txt`:

```cmake
cmake_minimum_required(VERSION 3.20)
project(TutoDialect LANGUAGES CXX)

find_package(MLIR REQUIRED CONFIG)

list(APPEND CMAKE_MODULE_PATH "${MLIR_CMAKE_DIR}")
include(TableGen)
include(AddMLIR)

add_subdirectory(include)
add_subdirectory(lib)

enable_testing()
add_subdirectory(test)
```

# Additional CMakeLists for Out-of-Tree Dialect

---

## `include/CMakeLists.txt`

```cmake
# Configure include directory for TableGen generated headers
add_subdirectory(TutoDialect)
```

---

## `include/TutoDialect/CMakeLists.txt`

```cmake
set(LLVM_TARGET_DEFINITIONS TutoDialect.td)
mlir_tablegen(TutoDialectDialect.h.inc -gen-dialect-decls)
mlir_tablegen(TutoDialectDialect.cpp.inc -gen-dialect-defs)
add_public_tablegen_target(TutoDialectIncGen)

set(LLVM_TARGET_DEFINITIONS TutoDialectOps.td)
mlir_tablegen(TutoDialectOps.h.inc -gen-op-decls)
mlir_tablegen(TutoDialectOps.cpp.inc -gen-op-defs)
add_public_tablegen_target(TutoDialectOpsIncGen)
```

---

## `lib/CMakeLists.txt`

```cmake
add_mlir_dialect_library(TutoDialect
  TutoDialect.cpp

  DEPENDS
  TutoDialectIncGen
  TutoDialectOpsIncGen

  LINK_LIBS PUBLIC
  MLIRIR
)
```

---

## `test/CMakeLists.txt`

```cmake
add_subdirectory(TutoDialect)
```

---

## `test/TutoDialect/CMakeLists.txt`

```cmake
configure_lit_site_cfg(
  ${CMAKE_CURRENT_SOURCE_DIR}/lit.site.cfg.py.in
  ${CMAKE_CURRENT_BINARY_DIR}/lit.site.cfg.py
)

set(TUTO_DIALECT_TEST_DEPENDS
  FileCheck count not
  TutoDialect
)

add_lit_testsuite(check-tuto-dialect "Running the TutoDialect tests"
  ${CMAKE_CURRENT_BINARY_DIR}
  DEPENDS ${TUTO_DIALECT_TEST_DEPENDS}
)
add_lit_testsuites(TUTO_DIALECT ${CMAKE_CURRENT_SOURCE_DIR} DEPENDS ${TUTO_DIALECT_TEST_DEPENDS})
```