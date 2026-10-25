---
layout: default
title: References
nav_order: 4
has_children: true
---

# API Reference

This section documents the public headers by namespace and type. All declarations are in namespace `math` unless a nested namespace is shown. The umbrella header is sufficient for most projects; use a module header when you want to include only a specific part of the library.

Use this section when you need exact signatures, defaults, dimensions, return types, or failure behavior.

| API | Reference page | Header |
|---|---|---|
| Conventions and status codes | [API conventions]({{ site.baseurl }}/reference/conventions.html) | All public headers |
| `math` root | [Constants, arithmetic, and conversion]({{ site.baseurl }}/reference/arithmetic.html) | `mathlib/arithmetic/arithmetic.hpp`, `mathlib/abstract/object.hpp` |
| `math::control` | [PID controller]({{ site.baseurl }}/reference/control.html) | `mathlib/control/pid.hpp` |
| `math::filters::KalmanFilter` | [Kalman filter]({{ site.baseurl }}/reference/kalman-filter.html) | `mathlib/filtering/kalman.hpp` |
| `math::filters::ExtendedKalmanFilter` | [Extended Kalman filter]({{ site.baseurl }}/reference/extended-kalman-filter.html) | `mathlib/filtering/ekf.hpp` |
| `math::filters::GaussianMixtureFilter` | [Gaussian mixture filter]({{ site.baseurl }}/reference/gaussian-mixture-filter.html) | `mathlib/filtering/gmm.hpp` |
| `math::filters::mcl` | [Particle filter]({{ site.baseurl }}/reference/particle-filter.html) | `mathlib/filtering/mcl/particle_filter.hpp` |
| `math::linalg` | [Matrix, vector, and Gaussian]({{ site.baseurl }}/reference/linear-algebra.html) | `mathlib/linalg/matrix.hpp`, `mathlib/linalg/vector.hpp`, `mathlib/linalg/gaussian.hpp` |
| `math::solver` | [Solvers]({{ site.baseurl }}/reference/solver.html) | `mathlib/linalg/solve_models.hpp` |
| `math::random` | [Random numbers]({{ site.baseurl }}/reference/random.html) | `mathlib/arithmetic/random.hpp` |
| Geometry | [Points, circles, and angles]({{ site.baseurl }}/reference/geometry.html) | `mathlib/geometry/geometry.hpp` |
| Glossary | [Glossary]({{ site.baseurl }}/reference/glossary.html) | — |

For the complete header map and package layout, see [Public API layout]({{ site.baseurl }}/development/public-api-layout.html).
