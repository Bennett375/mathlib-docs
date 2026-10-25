---
layout: default
title: Getting Started
nav_order: 2
has_children: true
---

# Getting Started

MathLib is a reusable C++ math library for PROS 4 projects. It provides numerical tools for feedback control, state estimation, field geometry, linear algebra, equation solving, and simulation.

MathLib is a library, not a complete robot program. The `src/main.cpp` file is the standard PROS demo entry point; robot-specific code belongs in your team's PROS project and calls MathLib through its public headers. This documentation is split into task-oriented guides and a detailed reference for the public API.

## At a glance

- **Language and target:** C++11 or newer; packaged for PROS 4, with a math core that does not call VEX hardware APIs.
- **Primary uses:** mechanism control, sensor fusion, localization, geometry, and numerical computation.
- **Error reporting:** filter operations report failures through `math::solver::SolveStatus`; factorization and solve calls return result objects that carry a status. Check it before relying on a result.
- **Package:** `mathlib`, version `1.0.0`.
- **Public entry point:** `#include "mathlib/mathlib.hpp"`.
- **Full reference:** The public API is documented in [API reference]({{ site.baseurl }}/reference/index.html).

## How the pieces fit together

A typical integration places MathLib between the sensor/model code and the robot control loop:

```text
sensor readings ──► state estimator ──► controller ──► motor commands
        ▲                   │                 │
        └──── motion model ─┴──── robot motion┘
```

- **Filtering** estimates the robot's state and uncertainty from models and measurements.
- **Control** turns the difference between a target and the current state into an output.
- **Geometry** represents field positions, distances, circles, and angles.
- **Linear algebra and solvers** provide the numerical operations used by estimators and other algorithms.
- **Arithmetic and random utilities** support common calculations and simulation.

MathLib does not read sensors or command motors itself. Your robot code supplies measurements, control inputs, and models.

## Choose a starting point

| Task | Start here | Example |
|---|---|---|
| Hold a lift or arm at a target | `math::control::PIDController` | `lift.update(target - position, dt)` |
| Fuse a linear motion model with sensor readings | `math::filters::KalmanFilter` | Predict position, then correct it with a distance measurement |
| Estimate a nonlinear or rotating system | `math::filters::ExtendedKalmanFilter` | Use a nonlinear heading-and-position model with Jacobians |
| Maintain multiple Gaussian hypotheses | `math::filters::GaussianMixtureFilter` | Keep separate position hypotheses when the measurement is ambiguous |
| Localize with sampled state hypotheses | `math::filters::mcl::ParticleFilter` | Score candidate robot poses against sensor data |
| Represent field positions and circles | `math::Point`, `math::Circle` | Measure distance between two field coordinates |
| Calculate circle intersections | `math::circle_intersections()` | Find candidate positions from two distance constraints |
| Solve a linear system | `math::solver::Solver<double>` | Solve `A * x = b` |
| Represent a Gaussian distribution | `math::linalg::Gaussian` | Score a candidate state using `log_pdf()` |
| Generate repeatable random samples | `math::random::Rng` | Add Gaussian noise to simulated sensor data |

## Next steps

- [Install the library]({{ site.baseurl }}/getting-started/installation.html)
- [Create your first program]({{ site.baseurl }}/getting-started/quick-start.html)
- [Browse the guides]({{ site.baseurl }}/guides/index.html)
- [Browse the API reference]({{ site.baseurl }}/reference/index.html)
