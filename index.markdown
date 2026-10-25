---
layout: page
title: Home
nav_order: 1
permalink: /
---

# Mathlib

A comprehensive mathematics library for Vex V5. It provides numerical tools for feedback control, state estimation, field geometry, linear algebra, equation solving, and simulation.

MathLib is a reusable C++ math library for PROS 4 projects. It is a library, not a complete robot program: your robot code supplies sensors, motors, models, and control-loop timing.

**Version:** `1.0.0` &nbsp;&nbsp; **Language:** C++11 or newer &nbsp;&nbsp; **Target:** PROS 4

[Get started]({{ site.baseurl }}/getting-started/getting-started.html){: .btn .btn-primary }
[Browse the API]({{ site.baseurl }}/reference/index.html){: .btn }

---

## Explore the documentation

### Getting started

Install the library, configure your project, and build your first program.

[Installation and setup]({{ site.baseurl }}/getting-started/installation.html)

[Quick start]({{ site.baseurl }}/getting-started/quick-start.html)

### Guides

Learn how to use the library's components in practical robotics applications.

[Browse guides]({{ site.baseurl }}/guides/index.html)

### API reference

Find documented interfaces, parameters, return values, and usage examples for each public API.

[Browse the reference]({{ site.baseurl }}/reference/index.html)

### Development

Understand the design, build process, public header layout, and performance considerations.

[Explore development]({{ site.baseurl }}/development/index.html)

---

The public entry point for every module is:

```cpp
#include "mathlib/mathlib.hpp"
```
