---
layout: default
title: Public API Layout
parent: Development Summary
nav_order: 2
---

# Public API Layout

The public API is organized by namespace under `include/mathlib/`. Implementation files under `src/mathlib/` are not part of the API contract.

| Path | Contents |
|---|---|
| `include/mathlib/mathlib.hpp` | Umbrella header for public modules |
| `include/mathlib/control/` | PID controller |
| `include/mathlib/filtering/` | Kalman, EKF, and Gaussian mixture filters |
| `include/mathlib/filtering/mcl/` | Particle filter, particle type, and Gaussian motion model |
| `include/mathlib/geometry/` | Point, circle, and geometry helpers |
| `include/mathlib/linalg/` | Matrix, vector, Gaussian distribution, and solver models |
| `include/mathlib/arithmetic/` | Arithmetic helpers and random source |
| `include/mathlib/abstract/` | `MathObj` base and conversion customization point |
| `src/mathlib/` | Library implementation; not part of the public API |
| `src/main.cpp` | Standard PROS demo shell; not the library API |
| `bin/mathlib.a` | Built static library in the described package layout |

## Header map

For a smaller include, use the module header directly. All paths below are relative to `include/`.

| Feature | Header |
|---|---|
| Umbrella header | `mathlib/mathlib.hpp` |
| PID controller | `mathlib/control/pid.hpp` |
| Kalman filter | `mathlib/filtering/kalman.hpp` |
| Extended Kalman filter | `mathlib/filtering/ekf.hpp` |
| Gaussian mixture filter | `mathlib/filtering/gmm.hpp` |
| Particle filter | `mathlib/filtering/mcl/particle_filter.hpp` |
| Particle type | `mathlib/filtering/mcl/particle.hpp` |
| Gaussian motion model | `mathlib/filtering/mcl/motion_model.hpp` |
| Matrix and vector | `mathlib/linalg/matrix.hpp` and `mathlib/linalg/vector.hpp` |
| Gaussian distribution | `mathlib/linalg/gaussian.hpp` |
| Solvers and status types | `mathlib/linalg/solve_models.hpp` |
| Geometry | `mathlib/geometry/geometry.hpp` |
| Arithmetic helpers | `mathlib/arithmetic/arithmetic.hpp` |
| Random source | `mathlib/arithmetic/random.hpp` |
| `MathObj` and conversions | `mathlib/abstract/object.hpp` |
