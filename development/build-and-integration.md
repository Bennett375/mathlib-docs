---
layout: default
title: Build and Integration
parent: Development Summary
nav_order: 3
---

# Build and Integration

## Build workflow

Use the normal PROS build workflow for this project, or run `make` in the repository root. The default `quick` goal performs an incremental build. Use `make clean` for a clean rebuild.

The library itself is a static archive. It is not header-only. Code that uses MathLib must include the public headers and link the built `mathlib` archive.

## Use from another PROS project

Choose one of these integration styles:

1. **Install the PROS library package.** Build this repository as a library, then install the resulting `mathlib` package into the other PROS project through the PROS library workflow. The package carries the public headers and static library.
2. **Add the headers and archive manually.** Copy or reference `include/mathlib/` from this project's repository, add the repository's `include/` directory to the other project's include path, and link `bin/mathlib.a`.
3. **Compile the sources directly.** Add `include/` to the include path and compile the `.cpp` files under `src/mathlib/` as part of the other project. Do not compile `src/main.cpp` into the library; it is the demo shell.

With any integration style, the public include line is:

```cpp
#include "mathlib/mathlib.hpp"
```

## Desktop and simulation use

The MathLib core does not include PROS headers or call VEX hardware APIs. You can compile its source files with a standard C++ toolchain for desktop tests and simulations. The project uses C++11 features and should be built with C++11 or newer.

For desktop builds, provide the same include path (`include/`) and link the MathLib source files or archive. The PROS kernel and VEX firmware libraries are not required by the math layer itself, but they are required for the robot target.

## Versioning and dependencies

The package version is `1.0.0` in the Makefile. The library depends on the C++ standard library and, for the robot target, the PROS 4 kernel. It has no other third-party library dependency.
