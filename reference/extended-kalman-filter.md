---
layout: default
title: Extended Kalman Filter
parent: References
nav_order: 5
---

# Extended Kalman Filter

`ExtendedKalmanFilter()` creates an uninitialized filter. It inherits `KalmanFilter`; use the inherited `initialize`, `reset`, `distribution`, `state`, `covariance`, `initialized`, and `state_dim` methods. The base linear `predict` and `update` methods remain available. For the shared matrix shapes, see [Filter and matrix shapes]({{ site.baseurl }}/reference/conventions.html#filter-and-matrix-shapes).

## Callback types

| Type | Exact signature | Meaning |
|---|---|---|
| `StateFunction` | `linalg::Vector<double>(const linalg::Vector<double>& x, const linalg::Vector<double>& u)` | Nonlinear motion model `f(x, u)` |
| `StateJacobian` | `linalg::Matrix<double>(const linalg::Vector<double>& x, const linalg::Vector<double>& u)` | Jacobian `df/dx` |
| `MeasurementFunction` | `linalg::Vector<double>(const linalg::Vector<double>& x)` | Nonlinear measurement model `h(x)` |
| `MeasurementJacobian` | `linalg::Matrix<double>(const linalg::Vector<double>& x)` | Jacobian `dh/dx` |
| `ResidualFunction` | `linalg::Vector<double>(const linalg::Vector<double>& measurement, const linalg::Vector<double>& predicted_measurement)` | Optional custom residual |

## Methods

| Method | Returns | Behavior |
|---|---|---|
| `predict_nonlinear(const StateFunction& f, const StateJacobian& F, const linalg::Vector<double>& control, const linalg::Matrix<double>& Q)` | `SolveStatus` | Evaluates `f` and `F` at the current state, then propagates the estimate and covariance |
| `predict_nonlinear(const StateFunction& f, const StateJacobian& F, const linalg::Matrix<double>& Q)` | `SolveStatus` | Same, but passes an empty control vector to the callbacks |
| `update_nonlinear(const linalg::Vector<double>& measurement, const MeasurementFunction& h, const MeasurementJacobian& H, const linalg::Matrix<double>& R, const ResidualFunction& residual = ResidualFunction())` | `SolveStatus` | Evaluates `h` and `H`, computes an innovation, then applies a Kalman correction |

## Behavior and failure notes

- Required callbacks (`f`/`F` for prediction, `h`/`H` for update) must be non-empty; a missing required callback returns `InvalidDimensions`. The residual callback is optional.
- The filter evaluates the nonlinear function at the current estimate and linearizes around that estimate. It does not compute numerical Jacobians; you supply them.
- The no-control prediction overload passes a zero-length control vector. A state function that assumes a control must handle that case.
- The default residual is `measurement - h(x)`. Supply a custom residual for wrapped angles or other nonlinear residual spaces.
- The custom residual is a full vector of length `m`, not a scalar. Use `math::angle_error` for heading residuals.
- The state function must return `n` elements, and `F` and `Q` must be `n x n`. The measurement function must return `m` elements, `H` must be `m x n`, and `R` must be `m x m`.
- A failed nonlinear prediction or update returns a status and leaves the previous distribution intact.

## Example

{% raw %}
```cpp
const double dt = 0.02;

auto f = [dt](const math::linalg::vec& x,
              const math::linalg::vec& u) {
    return math::linalg::vec{
        x(0) + x(1) * dt + 0.5 * u(0) * dt * dt,
        x(1) + u(0) * dt
    };
};

auto F = [dt](const math::linalg::vec&,
              const math::linalg::vec&) {
    return math::linalg::Matrix<double>{{1.0, dt}, {0.0, 1.0}};
};

auto h = [](const math::linalg::vec& x) {
    return math::linalg::vec{x(0)};
};

auto H = [](const math::linalg::vec&) {
    return math::linalg::Matrix<double>{{1.0, 0.0}};
};

math::filters::ExtendedKalmanFilter ekf;
math::linalg::vec x0{0.0, 0.0};
math::linalg::Matrix<double> P0{
    {1.0, 0.0},
    {0.0, 1.0}
};
ekf.initialize(x0, P0);

math::linalg::Matrix<double> Q{
    {0.001, 0.0},
    {0.0, 0.01}
};
math::linalg::Matrix<double> R{{0.25}};
math::linalg::vec control{0.5};
math::linalg::vec measurement{12.3};

ekf.predict_nonlinear(f, F, control, Q);
ekf.update_nonlinear(measurement, h, H, R);
```
{% endraw %}

## Angle residuals

For a heading measurement, a custom residual can look like:

```cpp
auto angle_residual =
    [](const math::linalg::vec& measurement,
       const math::linalg::vec& predicted) {
        return math::linalg::vec{
            math::angle_error(measurement(0), predicted(0))
        };
    };
```
