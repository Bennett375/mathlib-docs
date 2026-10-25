---
layout: default
title: Particle Filter
parent: References
nav_order: 7
---

# Particle Filter

`math::filters::mcl::ParticleFilter` represents a distribution using particles. Each particle contains a state vector and a weight. A motion callback advances each particle, and a log-likelihood callback scores how well its state explains a measurement. Resampling draws a new population with greater representation from higher-weight particles. For a task-oriented introduction, see the [state estimation guide]({{ site.baseurl }}/guides/state-estimation.html).

## `Particle`

| Member | Type | Meaning |
|---|---|---|
| `state` | `linalg::Vector<double>` | Particle state |
| `weight` | `double` | Relative weight |

Constructors:

- `Particle(std::size_t dim)` creates a zero state of size `dim` and weight `0.0`.
- `Particle(const linalg::Vector<double>& x, double w)` copies a state and weight.
- `as<std::string>()` is available through `MathObj<Particle>`.

## `ParticleFilterOptions`

| Field | Type | Default | Meaning |
|---|---|---:|---|
| `particle_count` | `std::size_t` | `500` | Required number of particles; must be greater than zero |
| `resample_ess_ratio` | `double` | `0.5` | Resample when effective sample size falls below this fraction of the population; must be finite and in `[0, 1]` |
| `min_weight` | `double` | `0.0` | Optional weight floor; must be finite, nonnegative, and less than `1 / particle_count`; `0` disables it |
| `resample_after_update` | `bool` | `true` | Automatically resample after a successful measurement update when the ESS threshold is crossed |

The constructor does not validate these values. `initialize` reports invalid options as `NumericalFailure`; `initialize_random` reports them as `InvalidDimensions`.

## `ParticleEstimate`

| Field or method | Type | Meaning |
|---|---|---|
| `status` | `math::solver::SolveStatus` | Result status; defaults to `NumericalFailure` |
| `mean` | `linalg::Vector<double>` | Weighted mean |
| `covariance` | `linalg::Matrix<double>` | Weighted covariance |
| `effective_sample_size` | `double` | ESS of the population used for the estimate |
| `success()` | `bool` | True when `status == Success` |

Constructor: `explicit ParticleEstimate(std::size_t n)` creates an `n`-dimensional result with zero mean, zero covariance, and zero effective sample size.

## Callback types

| Type | Exact signature | Meaning |
|---|---|---|
| `MotionModel` | `SolveStatus(const linalg::Vector<double>& state, const linalg::Vector<double>& control, random::RandomSource& random, linalg::Vector<double>& next_state)` | Advances one particle; write the next state through the output parameter |
| `LogLikelihoodModel` | `SolveStatus(const linalg::Vector<double>& state, const linalg::Vector<double>& measurement, double& log_likelihood)` | Returns the log probability of the measurement given the state through the output parameter |
| `InitialStateSampler` | `SolveStatus(random::RandomSource& random, linalg::Vector<double>& state)` | Produces one initial particle state |

## `ParticleFilter` methods

| Method | Returns | Behavior |
|---|---|---|
| `explicit ParticleFilter(random::RandomSource& random, const ParticleFilterOptions& options = ParticleFilterOptions())` | Filter | Stores a reference to the random source; the source must outlive the filter |
| `initialize(const std::vector<Particle>& particles)` | `SolveStatus` | Validates and copies an existing population |
| `initialize_random(std::size_t state_dimension, const InitialStateSampler& sampler)` | `SolveStatus` | Samples a new population of exactly `particle_count` particles |
| `predict(const linalg::Vector<double>& control, const MotionModel& motion_model)` | `SolveStatus` | Advances every particle through the motion callback |
| `update(const linalg::Vector<double>& measurement, const LogLikelihoodModel& likelihood_model)` | `SolveStatus` | Scores and reweights particles, then optionally resamples |
| `normalize_weights()` | `SolveStatus` | Normalizes the current weights and applies `min_weight` |
| `resample()` | `SolveStatus` | Performs systematic resampling and assigns equal weights |
| `ParticleEstimate estimate() const` | Estimate | Weighted mean, covariance, and ESS |
| `double effective_sample_size() const` | ESS | Returns `0.0` when uninitialized, empty, or the weight-square sum is invalid; otherwise computes `1 / sum(weight^2)` |
| `std::size_t particle_count() const` | Count | Actual current particle count |
| `std::size_t state_dim() const` | Dimension | `0` when uninitialized |
| `const std::vector<Particle>& particles() const` | Population | Read-only access |
| `void reset()` | Nothing | Clears the population and returns to the uninitialized state |

**Behavior and failure notes**

- `initialize` requires exactly `options.particle_count` particles, all with the same nonzero dimension, finite states, finite nonnegative weights, and a positive total weight. Weights are normalized before the population is committed.
- `initialize_random` requires a nonzero dimension and a non-empty sampler. It creates exactly `particle_count` particles with uniform initial weights. Each sampled state must have the requested size and contain only finite values.
- `predict` requires an initialized filter, a non-empty motion callback, and a finite control vector. The same control and callback are used for every particle. The callback receives the current state, control, random source, and an output state. It must write a state with the same dimension and finite values. The population is committed only if every particle succeeds.
- `update` requires an initialized filter, a non-empty likelihood callback, and a finite measurement. The same measurement and callback are used for every particle. The callback must write a log likelihood. Negative infinity is accepted as a zero likelihood; NaN and positive infinity are rejected.
- The update uses log-sum-exp normalization so very small likelihoods do not underflow the entire population. If every particle receives zero posterior weight, the update fails with `NumericalFailure`.
- After a successful update, the filter resamples when `resample_after_update` is true and the ESS is below `resample_ess_ratio * particle_count`.
- The measurement update is committed before automatic resampling. If resampling then fails, the returned status reports that failure even though the updated weights are already in place.
- `normalize_weights()` is public and applies the same `min_weight` floor as the update path. The public API exposes particles only through a const reference, so the filter's own methods normally handle weight changes.
- `resample()` uses systematic resampling with one uniform offset. It assigns equal weights and applies the configured floor. It does not alter particle states except by selecting existing particles.
- `min_weight` is a floor, not a guarantee of particle diversity. A floor that is too high can hide weight collapse; a floor at or above `1 / particle_count` is rejected by option validation.
- The motion model must add process noise itself if you want noisy particles. The filter does not add noise automatically. `GaussianMotionModel` is a helper that does.
- The likelihood callback must return a log likelihood, not a raw likelihood. Use `std::log` on a probability density or a negative squared error, as appropriate for your sensor model.
- The filter has no time-step parameter; include time in the control vector or the callback capture.
- `particles()` returns a reference that is invalidated by the next `initialize`, `initialize_random`, prediction, update, resample, or reset. Copy data before retaining it.
- `estimate()` returns `NumericalFailure` when the filter is uninitialized, the population is empty or malformed, or the current weights are invalid.

```cpp
math::random::Rng random(1234);

math::filters::mcl::ParticleFilterOptions options;
options.particle_count = 300;
options.resample_ess_ratio = 0.5;

math::filters::mcl::ParticleFilter filter(random, options);

auto sampler = [](math::random::RandomSource& source,
                  math::linalg::vec& state) {
    state = math::linalg::vec{
        100.0 * source.uniform01(),
        100.0 * source.uniform01(),
        math::TAU * source.uniform01()
    };
    return math::solver::SolveStatus::Success;
};

filter.initialize_random(3, sampler);
```

## `GaussianMotionModel`

`GaussianMotionModel` adapts a deterministic transition function plus Gaussian process noise to the `ParticleFilter::MotionModel` signature.

| Member | Returns | Behavior |
|---|---|---|
| `GaussianMotionModel(std::size_t state_dimension, std::size_t control_dimension, const TransitionFunction& transition, const linalg::Matrix<double>& covariance)` | Model | Factorizes the covariance during construction |
| `solver::SolveStatus status() const` | Construction status | Reports why construction failed |
| `bool valid() const` | Flag | True when construction succeeded |
| `std::size_t state_dim() const` | Dimension | Configured state dimension |
| `std::size_t control_dim() const` | Dimension | Configured control dimension |
| `const linalg::Matrix<double>& covariance() const` | Covariance | Process-noise covariance |
| `operator()(const linalg::Vector<double>& state, const linalg::Vector<double>& control, random::RandomSource& random, linalg::Vector<double>& next_state) const` | `SolveStatus` | Applies the transition, adds correlated Gaussian noise, and writes the next state |

The transition callback type is `SolveStatus(const linalg::Vector<double>& state, const linalg::Vector<double>& control, linalg::Vector<double>& predicted_state)`. It should produce a deterministic prediction and must write a state of the configured dimension; `GaussianMotionModel` adds the noise. Construction also requires a non-empty transition callback; `status()` reports the reason when construction fails.

Covariance requirements:

- `state_dimension` must be nonzero.
- Square, with one row per state dimension.
- Finite and approximately symmetric.
- Positive semidefinite in the same sense used by the factorizer: zero-variance directions are allowed, but materially negative pivots return `NotPositiveDefinite`.
- A singular covariance is acceptable here, unlike `Gaussian::create`, because this helper uses a semidefinite Cholesky factorization.

The model validates state and control dimensions and finite inputs on every call. If the model is invalid, the call returns the stored construction status. It does not modify the output state until the transition, noise draw, and finite checks all succeed.

Given a compatible two-dimensional particle filter:

```cpp
auto transition = [](const math::linalg::vec& state,
                     const math::linalg::vec& control,
                     math::linalg::vec& next) {
    next = math::linalg::vec{
        state(0) + control(0),
        state(1) + control(1)
    };
    return math::solver::SolveStatus::Success;
};

math::linalg::Matrix<double> process_noise{
    {0.01, 0.0},
    {0.0, 0.01}
};

math::filters::mcl::GaussianMotionModel model(
    2, 2, transition, process_noise);

math::linalg::vec control{1.0, 0.0};

if (model.valid()) {
    filter.predict(control, model);
}
```
