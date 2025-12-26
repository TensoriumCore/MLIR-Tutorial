This guide provides a complete walkthrough for setting up a minimal **MLIR out-of-tree dialect** project. It is aimed at developers who want to create and experiment with custom MLIR dialects outside the LLVM source tree, allowing for rapid iteration without modifying the main LLVM codebase. We will cover installation of LLVM/MLIR, project structure, and detailed CMake configuration. By the end of this tutorial, you will have a working project that builds your custom dialect and runs it through an `mlir-opt`-like executable.
# MLIR Out-of-Tree Dialect Setup Guide

This document explains in detail how to set up a minimal **MLIR out-of-tree dialect** project. It covers:

- Installing LLVM & MLIR
    
- Project structure
    
- CMake configuration for dialects & tools
    

---

## Installing LLVM and MLIR

To develop an **out-of-tree** MLIR dialect, you need LLVM and MLIR installed.

There are two common approaches:

- **Recommended**: Build LLVM/MLIR from source to ensure compatibility.
    
- Alternative: Use a prebuilt package (less flexible for MLIR development).
    

### **Build LLVM/MLIR from source**

```bash
git clone https://github.com/llvm/llvm-project.git
cd llvm-project

export PREFIX=/opt/local/libexec/llvm   # installation directory

mkdir build && cd build
cmake -G Ninja ../llvm \
  -DCMAKE_INSTALL_PREFIX=$PREFIX \
  -DLLVM_ENABLE_PROJECTS="mlir;clang;lld" \
  -DLLVM_TARGETS_TO_BUILD="X86" \        # ISA target (adjust as needed)
  -DLLVM_ENABLE_RTTI=ON \               # Needed for out-of-tree dialects
  -DMLIR_ENABLE_BINDINGS_PYTHON=OFF \   # Enable if using Pybind11
  -DCMAKE_BUILD_TYPE=Release

ninja -j64  # Adjust -j for your CPU
sudo ninja install
```

**Check installation:**

```bash
mlir-opt --version
```

It should print the expected MLIR version.

---

## Project Structure

Our project will be called `MLIR-Tutorial` with the following structure:

```
MLIR-Tutorial/
├── CMakeLists.txt
├── include/
│   └── TutoDialect/
│       ├── TutoDialect.h
│       ├── TutoDialectOps.h
│       ├── TutoDialect.td
│       └── TutoDialectOps.td
├── lib/
│   ├── CMakeLists.txt
│   └── TutoDialect.cpp
├── tuto-opt/
│   ├── CMakeLists.txt
│   └── Tuto-opt.cpp
├── test/
│   └── TutoTest.mlir
```

 **Explanation of folders:**

- **`include/`** → Public headers and `.td` files for dialect and operations.
    
- **`lib/`** → Dialect C++ implementation.
    
- **`tuto-opt/`** → Main executable (similar to `mlir-opt`).
    
- **`test/`** → MLIR test files.
    

---

## Root CMakeLists.txt

The root `CMakeLists.txt` sets up MLIR/LLVM paths and adds subdirectories.

```cmake
cmake_minimum_required(VERSION 3.20)
project(tensorium-dialect LANGUAGES CXX C)

set(LLVM_DIR "/opt/local/libexec/llvm-20/lib/cmake/llvm")
set(MLIR_DIR "/opt/local/libexec/llvm-20/lib/cmake/mlir")

set(CMAKE_CXX_STANDARD 17 CACHE STRING "C++ standard")

find_package(MLIR REQUIRED CONFIG)

set(LLVM_RUNTIME_OUTPUT_INTDIR ${CMAKE_BINARY_DIR}/bin)
set(LLVM_LIBRARY_OUTPUT_INTDIR ${CMAKE_BINARY_DIR}/lib)
set(MLIR_BINARY_DIR ${CMAKE_BINARY_DIR})

list(APPEND CMAKE_MODULE_PATH "${MLIR_CMAKE_DIR}")
list(APPEND CMAKE_MODULE_PATH "${LLVM_CMAKE_DIR}")
include(TableGen)
include(AddLLVM)
include(AddMLIR)
include(HandleLLVMOptions)

include_directories(${LLVM_INCLUDE_DIRS} ${MLIR_INCLUDE_DIRS} ${PROJECT_SOURCE_DIR}/include ${PROJECT_BINARY_DIR}/include)
link_directories(${LLVM_BUILD_LIBRARY_DIR})
add_definitions(${LLVM_DEFINITIONS})

set(LLVM_LIT_ARGS "-sv" CACHE STRING "lit default options")

add_subdirectory(include)
add_subdirectory(lib)
add_subdirectory(tuto-opt)
```

---

## CMake for Include Directory

**`include/CMakeLists.txt`**

```cmake
add_subdirectory(TutoDialect)
```

---

## CMake for Dialect TableGen

**`include/TutoDialect/CMakeLists.txt`**

```cmake
add_mlir_dialect(TutoOps tuto)
add_mlir_doc(TutoDialect TutoDialect Tuto/ -gen-dialect-doc)
add_mlir_doc(TutoOps TutoOps Tuto/ -gen-op-doc)
```

---

## CMake for Dialect Library

**`lib/Tuto/CMakeLists.txt`**

```cmake
add_mlir_dialect_library(MLIRTuto
  TutoDialect.cpp
  TutoOps.cpp
  DEPENDS
    MLIRTutoOpsIncGen
  LINK_LIBS PUBLIC
    MLIRIR
)
```

---

## CMake for Executable (`tuto-opt`)

**`tuto-opt/CMakeLists.txt`**

```cmake
get_property(dialect_libs GLOBAL PROPERTY MLIR_DIALECT_LIBS)
get_property(conversion_libs GLOBAL PROPERTY MLIR_CONVERSION_LIBS)
set(LIBS
    ${dialect_libs}
    ${conversion_libs}
    MLIRTuto
    MLIROptLib
)

add_llvm_executable(Tuto-opt Tuto-opt.cpp)
llvm_update_compile_flags(Tuto-opt)
target_link_libraries(Tuto-opt PRIVATE ${LIBS})
```

---

## Summary

1. **Install LLVM & MLIR** (build from source recommended)
    
2. **Create project structure** with `include`, `lib`, `tuto-opt`, `test`
    
3. **Configure CMake** for dialect, library, and executable
    
4. **Build the project**:
    

```bash
mkdir build && cd build
cmake -G Ninja ..
ninja
```

You should now be able to run:

```bash
./bin/Tuto-opt test/TutoTest.mlir
```