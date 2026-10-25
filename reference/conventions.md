---
layout: default
title: API Conventions
parent: References
nav_order: 1
---

# API Conventions

All declarations are in namespace `math` unless a nested namespace is shown. The umbrella header is `mathlib/mathlib.hpp`; use a module header when you want to include only a specific part of the library.

## Public API conventions

| Topic | Convention |
|---|---|
| Public header | `#include "mathlib/mathlib.hpp"` |
| Namespaces | `math`, `math::control`, `math::filters`, `math::filters::mcl`, `math::linalg`, `math::solver`, `math::random` |
| Angles | Geometry and angle helpers use radians; PID error units are caller-defined |
| Matrix indexing | Zero-based, row-major storage |
| Matrix/vector scalar types | `int`, `float`, and `double` are explicitly instantiated; use `double` for numerical work |
| High-level filters | `double` only |
| Time | PID `dt` is in seconds; filters use caller-defined discrete steps |
| Units | Caller-defined; keep one unit system across a model |
| Error reporting | Direct matrix/vector misuse throws; filter methods return `SolveStatus` or a result object, and factorization/solve calls return result objects that carry one |
| Thread safety | No internal locking; do not share an instance across tasks without synchronization |
| Returned references | Several accessors return const references; copy values before the next update if you need to retain them |
| Internal API | `math::solver::detail`, private members, and implementation files are not part of the stable public API |

## Failure styles

| API family | Failure style |
|---|---|
| Matrix and vector constructors/arithmetic | Throws `std::invalid_argument`, `std::out_of_range`, or `std::domain_error` |
| `Solver<T>::solve(A, b)` | Returns `SolveResult<T>` with a status |
| `Solver<T>::solve(A, B)` | Throws `std::invalid_argument` for a row mismatch; throws `std::runtime_error` if any column fails |
| `determinant` and `numerical_rank` | Return a fallback value; use the factorization APIs when you need the exact failure reason |
| PID controller | Returns the previous output for an invalid sample; it has no status result |
| Filters | State-changing methods return `math::solver::SolveStatus`; estimate helpers return result objects with a status. Check it before reading the estimate |
| Gaussian evaluation | Returns `GaussianValueResult`; check `success()` |
| Geometry | No general validation or status reporting; validate radii and inputs yourself |
| Random utilities | No error reporting; the caller owns distribution and range choices |

## Status codes

`math::solver::SolveStatus` is the shared status enum for filter methods and solver result objects.

| Status | Meaning | Example response |
|---|---|---|
| `Success` | Operation completed | Use the result |
| `InvalidDimensions` | Input shapes do not match | Check state, measurement, and matrix dimensions |
| `SingularMatrix` | A unique solution cannot be computed | Check the model or regularize the system |
| `NotPositiveDefinite` | A matrix expected to be positive definite is not | Check covariance symmetry and variances |
| `NoConvergence` | An iterative method did not converge within its limit | Review scaling, tolerance, and `max_iterations` |
| `NonFiniteInput` | An input contains NaN or infinity | Reject the sensor sample or fix upstream calculations |
| `NumericalFailure` | A valid numerical result could not be produced | Check units, scaling, and numerical conditioning |
| `UnsupportedMethod` | The selected method or required callback is not supported for the request | Check the method value and callback assignments |

Example status handling:

```cpp
auto status = filter.predict(F, Q);
if (status != math::solver::SolveStatus::Success) {
    // Log the status and decide whether to skip this cycle,
    // retain the previous estimate, or recover.
}
```

Check status codes before using numerical results. A result object may still contain a numeric field when its status indicates failure; only use the value after checking the status.

## Filter and matrix shapes

Filters share a common idea: a state vector describes the system, a covariance matrix describes uncertainty, a prediction step moves the estimate forward, and an update step corrects it with a measurement. Filter operations report failures through `math::solver::SolveStatus`, either directly or in a result object; pure accessors return values directly.

For the linear and extended Kalman filters, the conventional shapes are:

| Object | Shape | Meaning |
|---|---|---|
| State `x` | `n x 1` | Estimated state |
| Covariance `P` | `n x n` | State uncertainty |
| Transition `F` | `n x n` | Linear motion model |
| Process noise `Q` | `n x n` | Motion-model uncertainty |
| Control matrix `B` | `n x control_dim` | How a control input affects the state |
| Control `u` | `control_dim x 1` | Known input |
| Measurement `z` | `m x 1` | Sensor reading |
| Measurement matrix `H` | `m x n` | How state maps to measurement |
| Measurement noise `R` | `m x m` | Sensor uncertainty |

`Q`, `R`, and the initial covariance should be symmetric. The implementation checks symmetry with a small tolerance, and the covariance calculations require the resulting matrices to be positive definite. A zero-variance dimension is not a valid covariance for `Gaussian::create`; use a small positive variance instead.
