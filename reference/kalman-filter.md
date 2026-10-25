---
layout: default
title: Kalman Filter
parent: References
nav_order: 4
---

# Kalman Filter

`math::filters::KalmanFilter` is intended for linear state and measurement models. For the shared state, covariance, and measurement matrix shapes used by the linear and extended Kalman filters, see [Filter and matrix shapes]({{ site.baseurl }}/reference/conventions.html#filter-and-matrix-shapes).

## API

| Method | Returns | Behavior |
|---|---|---|
| `KalmanFilter()` | Uninitialized filter | No valid state or covariance is available until initialization succeeds |
| `initialize(const linalg::Gaussian& initial_distribution)` | `SolveStatus` | Validates and copies the distribution |
| `initialize(const linalg::Vector<double>& initial_state, const linalg::Matrix<double>& initial_covariance)` | `SolveStatus` | Builds a Gaussian from the state and covariance |
| `predict(const linalg::Matrix<double>& F, const linalg::Matrix<double>& Q)` | `SolveStatus` | `x' = F*x`, `P' = F*P*F^T + Q` |
| `predict(const linalg::Matrix<double>& F, const linalg::Matrix<double>& B, const linalg::Vector<double>& u, const linalg::Matrix<double>& Q)` | `SolveStatus` | `x' = F*x + B*u`, `P' = F*P*F^T + Q` |
| `update(const linalg::Vector<double>& measurement, const linalg::Matrix<double>& H, const linalg::Matrix<double>& R)` | `SolveStatus` | Corrects the estimate with `z = H*x + noise` |
| `const linalg::Gaussian& distribution() const` | Distribution | Full internal distribution |
| `const linalg::Vector<double>& state() const` | State | Shortcut for `distribution().mean()` |
| `const linalg::Matrix<double>& covariance() const` | Covariance | Shortcut for `distribution().covariance()` |
| `bool initialized() const` | Flag | Whether a valid initialization has completed |
| `std::size_t state_dim() const` | Dimension | `0` when uninitialized |
| `void reset()` | Nothing | Returns the filter to the uninitialized state |

**Behavior and failure notes**

- Initialization requires a non-empty state and a square, symmetric, positive-definite covariance. A failed initialization leaves an existing filter unchanged.
- `predict` requires `F` and `Q` to be `n x n`. The control overload also requires `B` to be `n x control_dim` and `u` to have `control_dim` elements.
- `update` requires a non-empty measurement, `H` of size `m x n`, and `R` of size `m x m`.
- All inputs to `predict` and `update` must be finite. `Q` and `R` must pass the covariance symmetry check; the innovation covariance and the resulting covariance must be positive definite.
- Initialization follows `Gaussian::create`, which checks the covariance but not the initial mean. Supply a finite initial state.
- The update uses a numerically stable Joseph covariance update and solves for the gain with Cholesky factorization. It does not form an explicit matrix inverse.
- A failed prediction or update returns a status and leaves the previous distribution intact.
- `F` encodes the time step. The filter has no separate `dt` parameter.
- The filter does not add a control input during `update`; put control effects in `predict`.
- `state()`, `covariance()`, and `distribution()` return references. Copy the values you need before the next filter operation.
- If an operation fails, the returned status is the source of truth. The accessors can still return the last successful estimate; do not treat it as the result of the failed call.

{% raw %}
```cpp
const double dt = 0.02;
math::filters::KalmanFilter filter;

math::linalg::vec x0{0.0, 0.0};
math::linalg::Matrix<double> P0{
    {1.0, 0.0},
    {0.0, 1.0}
};

if (filter.initialize(x0, P0) != math::solver::SolveStatus::Success) {
    // Handle initialization failure.
}

math::linalg::Matrix<double> F{
    {1.0, dt},
    {0.0, 1.0}
};
math::linalg::Matrix<double> Q{
    {0.001, 0.0},
    {0.0, 0.01}
};
math::linalg::Matrix<double> H{{1.0, 0.0}};
math::linalg::Matrix<double> R{{0.25}};

math::linalg::vec measurement{measured_distance};

if (filter.predict(F, Q) == math::solver::SolveStatus::Success) {
    if (filter.update(measurement, H, R) ==
        math::solver::SolveStatus::Success) {
        math::linalg::vec estimate = filter.state();
    }
}
```
{% endraw %}
