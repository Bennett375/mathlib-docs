---
layout: default
title: Performance and Thread Safety
parent: Development Summary
nav_order: 4
---

# Performance and Thread Safety

The V5 has limited compute and memory compared with a desktop. These notes describe the main costs and ownership rules to consider when choosing state dimensions, mixture sizes, and particle counts.

## Computational cost

The library uses dense matrices and ordinary dynamic allocation. The following table gives asymptotic estimates, not measured timings:

| Operation | Approximate cost |
|---|---|
| Matrix or vector addition/subtraction | `O(m * n)` |
| Matrix multiplication | `O(m * n * p)` |
| Transpose | `O(m * n)` |
| Determinant or inverse | `O(n^3)` |
| LU or Cholesky decomposition | `O(n^3)` |
| QR decomposition | `O(m * n^2)` for `m >= n` |
| Kalman prediction | `O(n^3)` |
| Kalman update | `O(n^3 + m^3 + n^2 * m)` |
| GMM prediction/update | Roughly `O(K)` times the corresponding Kalman cost, plus pruning |
| Particle prediction/update | `O(N)` callback calls, plus callback cost |
| Particle resampling | `O(N)` |

`n` is state dimension, `m` is measurement dimension, `K` is GMM component count, and `N` is particle count.

## Memory

- Every `Matrix` and `Vector` owns a `std::vector` of elements.
- Matrix operations such as multiplication and transpose allocate a new result.
- A Kalman filter stores its distribution and creates temporary matrices during prediction and update.
- A GMM stores up to `max_components` distributions, each with a mean and covariance.
- A particle filter stores `particle_count` state vectors and weights.
- There is no fixed-size matrix specialization, memory pool, or sparse matrix type.

On the robot, use the smallest state dimension that describes the problem, keep GMM component counts modest, and tune particle count against available CPU time. Avoid `inverse()` when a solve or factorization will do.

## Numerical choices

- Use `double` for filters, solvers, geometry, and PID work.
- Use `int` matrices and vectors only for exact integer work. Solver results for `int` are truncated when converted back from double.
- Keep covariance matrices symmetric. The filters symmetrize some intermediate covariances, but clearly asymmetric inputs can still fail validation.
- Prefer a small positive variance floor over an exact zero variance when constructing a Gaussian.
- Scale states and measurements so matrix entries are not wildly different in magnitude.
- Use `SolveOptions::check_finite` to catch bad data early during development; disable it only when the input path is already trusted and validated.

## Thread safety

MathLib does not provide internal locking. A filter, PID controller, or random source should be owned by one task at a time. If multiple tasks must share an object, add synchronization in your robot code.

`ParticleFilter` stores a reference to a `RandomSource`. The source must outlive the filter. Do not share an `Rng` between tasks without synchronization.

The library has no mutable global state in normal use. Individual objects still require synchronization if they are accessed or modified concurrently.
