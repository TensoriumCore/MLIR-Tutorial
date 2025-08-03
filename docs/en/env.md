
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
│   └── MyDialect/
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
project(MyDialect LANGUAGES CXX)

find_package(MLIR REQUIRED CONFIG)

list(APPEND CMAKE_MODULE_PATH "${MLIR_CMAKE_DIR}")
include(TableGen)
include(AddMLIR)

add_subdirectory(include)
add_subdirectory(lib)

enable_testing()
add_subdirectory(test)
```