---
layout: default
title: Gaussian Mixture Filter
parent: References
nav_order: 6
---

# Gaussian Mixture Filter

`math::filters::GaussianMixtureFilter` tracks several Gaussian hypotheses instead of representing uncertainty with only one Gaussian. This is useful when a sensor observation or field layout leaves multiple plausible states. For the shared matrix shapes, see [Filter and matrix shapes]({{ site.baseurl }}/reference/conventions.html#filter-and-matrix-shapes).

## Supporting types

`GaussianComponent`:

| Field | Type | Default | Meaning |
|---|---|---:|---|
| `distribution` | `linalg::Gaussian` | Invalid | One Gaussian hypothesis |
| `weight` | `double` | `0.0` | Relative hypothesis weight |

Its default constructor leaves the distribution invalid and the weight at `0.0`.

`GMMOptions`:

| Field | Type | Default | Meaning |
|---|---|---:|---|
| `max_components` | `std::size_t` | `8` | Maximum number of components retained after pruning; must be greater than zero |
| `min_weight` | `double` | `0.01` | Components below this normalized weight are removed; must be finite and nonnegative; if all would be removed, the strongest is kept |
| `merge_threshold` | `double` | `2.0` | Components merge when their average squared Mahalanobis distance is below this value; must be finite and nonnegative; `0` disables merging |
| `covariance_floor` | `double` | `1e-6` | Minimum diagonal variance applied during regularization; must be finite and nonnegative |

Invalid options, including `max_components == 0` or a negative/non-finite `min_weight`, `merge_threshold`, or `covariance_floor`, make `initialize` return `InvalidDimensions`.

`GMMEstimate`:

| Field or method | Type | Meaning |
|---|---|---|
| `status` | `math::solver::SolveStatus` | Result status; defaults to `NumericalFailure` |
| `mean` | `linalg::Vector<double>` | Combined weighted mean |
| `covariance` | `linalg::Matrix<double>` | Combined covariance, including between-component uncertainty |
| `success()` | `bool` | True when `status == Success` |

Constructor: `GMMEstimate(std::size_t n)` creates an `n`-dimensional estimate with zero mean and zero covariance.

## Methods

| Method | Returns | Behavior |
|---|---|---|
| `explicit GaussianMixtureFilter(const GMMOptions& options = GMMOptions())` | Filter | Stores options; does not validate them yet |
| `initialize(const std::vector<GaussianComponent>& components)` | `SolveStatus` | Validates, normalizes, and prunes a new component set |
| `predict(const linalg::Matrix<double>& F, const linalg::Matrix<double>& Q)` | `SolveStatus` | Applies `x' = F*x`, `P' = F*P*F^T + Q` to every component |
| `update(const linalg::Vector<double>& measurement, const linalg::Matrix<double>& H, const linalg::Matrix<double>& R)` | `SolveStatus` | Updates every component, reweights by measurement likelihood, then prunes |
| `prune()` | `SolveStatus` | Removes weak components, merges similar ones, and enforces `max_components` |
| `GMMEstimate estimate() const` | Combined estimate | Weighted mean and covariance of the mixture |
| `std::size_t component_count() const` | Count | Number of current components |
| `const std::vector<GaussianComponent>& components() const` | Component list | Read-only access |
| `const GaussianComponent& most_likely_component() const` | Strongest component | Returns a static invalid component when the mixture is empty |

**Behavior and failure notes**

- `initialize` requires at least one component, a nonzero common dimension, valid Gaussian distributions, finite nonnegative weights, and a positive total weight. Weights are normalized. A failure leaves the previous mixture intact.
- A component whose weight is below `min_weight` is removed during pruning, but the filter never removes every component solely because of the weight threshold.
- `predict` requires an initialized, non-empty mixture and `F` and `Q` of size `n x n`. The same `F` and `Q` apply to every component. `Q` is symmetrized and regularized with `covariance_floor`, then checked for positive definiteness. The prediction commits only if every component succeeds.
- `update` requires an initialized, non-empty mixture, a non-empty measurement, `H` of size `m x n`, and `R` of size `m x m`. The same `H` and `R` apply to every component. `R` is symmetrized and regularized with `covariance_floor` before use. Each component is updated with a Kalman-style correction, weighted by the innovation likelihood, and normalized with a log-sum-exp calculation.
- `update` commits the updated components before calling `prune()`. If pruning then fails, the returned status reports the pruning failure even though the measurement update is already in place.
- `prune()` removes components below `min_weight`, merges components whose average bidirectional squared Mahalanobis distance is below `merge_threshold`, keeps at most `max_components` of the largest weights, and renormalizes.
- `covariance_floor` is applied after symmetrization. It does not by itself guarantee positive definiteness; the Cholesky validation does.
- `estimate()` includes both the within-component covariance and the spread between component means, so it can describe a multimodal mixture with a large covariance.
- `most_likely_component()` returns a reference to a static invalid component when the mixture is empty. Check `component_count()` before using it.
- `components()` returns a reference that is invalidated by the next `initialize`, `predict`, `update`, or `prune`. Copy component data if you need to retain it.
- `estimate()` returns `InvalidDimensions` with a zero-dimensional result when the mixture is empty. It returns `NumericalFailure` if the total weight or the combined mean/covariance is not finite.
- GMM prediction has no `B`/`u` control overload. Add the control effect to the state model, augment the state, or reinitialize the mixture with shifted means.
- `GMMOptions` are fixed at construction; there is no setter for changing them later.
- To effectively disable mixture management, set `min_weight = 0`, `merge_threshold = 0`, and `max_components` at least as large as the number of components you intend to keep. Positive covariance regularization is still advisable.

Given a component vector and model matrices, a typical cycle is:

```cpp
math::filters::GMMOptions options;
options.max_components = 12;

math::filters::GaussianMixtureFilter filter(options);

if (filter.initialize(components) ==
    math::solver::SolveStatus::Success) {
    filter.predict(F, Q);
    filter.update(measurement, H, R);

    math::filters::GMMEstimate estimate = filter.estimate();
    if (estimate.success()) {
        math::linalg::vec mean = estimate.mean;
    }
}
```
