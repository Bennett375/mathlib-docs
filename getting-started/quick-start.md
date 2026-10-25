---
layout: default
title: Quick Start
parent: Getting Started
nav_order: 2
---

# Quick Start

## Include the library

For most projects, include the umbrella header in a source file:

```cpp
#include "mathlib/mathlib.hpp"
```

This includes the public arithmetic, random, control, filtering, geometry, and linear-algebra headers. For example, a mechanism controller can be created and updated like this:

```cpp
math::control::PIDController lift(
    math::control::PIDGains(1.2, 0.0, 0.08));

double error = target_position - current_position;
double motor_command = lift.update(error, 0.02); // dt in seconds
```

Depend on the public headers rather than implementation files. Public headers are under `include/mathlib/`; implementation files and build artifacts are not part of the API contract.

## Quick-start checklist

- [ ] Include `mathlib/mathlib.hpp`.
- [ ] Choose the module that matches the problem.
- [ ] Define consistent units and matrix dimensions.
- [ ] Check statuses before consuming filter or solver results.
- [ ] Log state, covariance, error, and sensor values during testing.
- [ ] Use a fixed non-zero random seed for repeatable simulation tests.
- [ ] Keep hardware-specific code outside the math layer.
- [ ] Validate all models and controller settings on the robot in a controlled test.

## Where to go next

- [PID control guide]({{ site.baseurl }}/guides/control.html)
- [State estimation guide]({{ site.baseurl }}/guides/state-estimation.html)
- [Task recipes]({{ site.baseurl }}/guides/task-recipes.html)
- [API reference]({{ site.baseurl }}/reference/index.html)
