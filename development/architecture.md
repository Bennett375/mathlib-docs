---
layout: default
title: Architecture
parent: Development Summary
nav_order: 1
---

# Architecture

MathLib separates reusable numerical code from the robot application. The math core does not read sensors or command motors; robot code supplies measurements, control inputs, models, and timing, then consumes the numerical results.

## Layers

| Layer | Main API | Responsibility |
|---|---|---|
| Root utilities | `math::PI`, `math::clamp`, `math::MathObj` | Constants, scalar helpers, and value conversion |
| Linear algebra | `math::linalg::Matrix`, `Vector`, and `Gaussian` | Dense storage, matrix/vector operations, and Gaussian distributions |
| Solvers | `math::solver` | LU, Cholesky, QR, and conjugate-gradient solving with shared status codes |
| State estimation | `math::filters` | Linear and nonlinear Kalman filters, Gaussian mixtures, and particle filters |
| Control | `math::control::PIDController` | Feedback control for mechanisms |
| Geometry | `math::Point`, `math::Circle`, and angle helpers | Field coordinates, distances, intersections, and wrapped angles |
| Randomness | `math::random::RandomSource` and `Rng` | Uniform and normal samples for simulation and particle filters |

The layers build on each other. Filters use `linalg` distributions and `solver` statuses; solvers use dense `linalg` matrices; and robot-facing guides combine the layers for tasks such as localization and mechanism control.

## Data flow

A typical control cycle follows this path:

1. Read a sensor or estimate the current state.
2. Apply the application's motion and measurement models.
3. Run a filter predict/update cycle, checking the returned status.
4. Compute an error from the target state.
5. Run the controller and map its output to safe actuator commands.
6. Log the state, covariance, error, and timing needed for tuning.

The application owns this loop. MathLib does not schedule tasks, allocate hardware, or impose a control period.

## Design boundaries

- The public API is defined by the headers under `include/mathlib/`; implementation files and `math::solver::detail` are internal.
- Matrices and vectors are dense value types. There is no sparse matrix or fixed-size specialization.
- High-level filters use `double`; matrix, vector, and solver templates also support `int` and `float`.
- Failures are reported through return values and status codes in high-level code. Direct matrix/vector misuse can throw standard exceptions.
- Objects hold no internal locks. Share a filter, controller, or random source between tasks only with external synchronization.
- Use the [public API layout]({{ site.baseurl }}/development/public-api-layout.html) for the header map and the [API conventions]({{ site.baseurl }}/reference/conventions.html) for status and shape rules.
