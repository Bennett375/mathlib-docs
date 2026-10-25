---
layout: default
title: State Estimation
parent: Guides
nav_order: 2
---

# State Estimation

Filters combine a model of how a system moves with noisy measurements. A filter's **state** might contain position and velocity; its **covariance** represents uncertainty in that estimate. Prediction advances the estimate using the model, and the measurement update corrects it according to the model and sensor uncertainty.

A typical cycle is:

1. Initialize the estimate and its covariance.
2. Predict the next state using the motion model.
3. Correct the prediction with a new measurement.
4. Check each returned status, then read the estimate.

## KalmanFilter

`math::filters::KalmanFilter` is intended for linear state and measurement models.

- `initialize(state, covariance)` initializes the state distribution.
- `predict(F, Q)` predicts using the state transition matrix `F` and process-noise covariance `Q`.
- `predict(F, B, u, Q)` also includes a known control input `u` through matrix `B`.
- `update(measurement, H, R)` corrects the estimate using measurement matrix `H` and measurement-noise covariance `R`.
- `state()`, `covariance()`, and `distribution()` expose the current estimate.
- `reset()` clears the filter's initialized state.

Example for a two-element state vector:

```cpp
math::filters::KalmanFilter filter;

math::linalg::vec initial_state{0.0, 0.0};
auto initial_covariance =
    math::linalg::Matrix<double>::identity(2) * 10.0;

auto status = filter.initialize(initial_state, initial_covariance);
if (status == math::solver::SolveStatus::Success) {
    // Example model: a 2-state system with identity transition.
    auto F = math::linalg::Matrix<double>::identity(2);
    auto Q = math::linalg::Matrix<double>::identity(2) * 0.1;
    auto H = math::linalg::Matrix<double>::identity(2);
    auto R = math::linalg::Matrix<double>::identity(2) * 0.5;
    math::linalg::vec measurement{1.0, 0.2};

    status = filter.predict(F, Q);
    if (status == math::solver::SolveStatus::Success) {
        status = filter.update(measurement, H, R);
    }
    if (status == math::solver::SolveStatus::Success) {
        math::linalg::vec estimate = filter.state();
    }
}
```

The matrices in a real robot model must match your state definition and measurement units. The example demonstrates the call sequence; it is not a complete drivetrain model.

**Understanding `Q`, `R`, and initial covariance**

- `Q` describes uncertainty in the motion model. Increasing it generally makes the filter less confident in its prediction.
- `R` describes uncertainty in the measurement. Increasing it generally makes the filter trust that measurement less.
- Initial covariance describes uncertainty in the starting state. Larger values mean the initial estimate is less certain.

If a prediction or update fails, inspect the returned status before using the estimate.

## ExtendedKalmanFilter

`math::filters::ExtendedKalmanFilter` extends the Kalman filter for nonlinear motion or measurement models. You provide the functions and their Jacobians:

- `StateFunction` models the next state, `f(x, u)`.
- `StateJacobian` provides the derivative of the state function with respect to the state.
- `MeasurementFunction` models the expected sensor reading, `h(x)`.
- `MeasurementJacobian` provides the derivative of the measurement function.
- An optional `ResidualFunction` calculates the measurement residual; use it for special handling such as wrapped angles.
- `predict_nonlinear()` performs a nonlinear prediction, with or without an explicit control vector.
- `update_nonlinear()` performs a nonlinear measurement update.

Example callback setup for a simple state model:

```cpp
math::filters::ExtendedKalmanFilter filter;

// Example state transition: x_next = x + u.
// This simple example uses the state vector and control vector directly.
math::filters::ExtendedKalmanFilter::StateFunction f =
    [](const math::linalg::vec& x, const math::linalg::vec& u) {
        return x + u;
    };

math::filters::ExtendedKalmanFilter::StateJacobian F =
    [](const math::linalg::vec& x, const math::linalg::vec&) {
        return math::linalg::Matrix<double>::identity(x.size());
    };
```

After initializing the filter, call `predict_nonlinear(f, F, control, Q)`. For a real nonlinear model, define the state function and Jacobian consistently, then provide matching measurement functions and Jacobians to `update_nonlinear()`.

## GaussianMixtureFilter

`math::filters::GaussianMixtureFilter` tracks several Gaussian hypotheses instead of representing uncertainty with only one Gaussian. This is useful when a sensor observation or field layout leaves multiple plausible states.

Each `GaussianComponent` contains a `linalg::Gaussian` distribution and a `weight`.

```cpp
math::linalg::Gaussian first_distribution;
math::linalg::Gaussian second_distribution;

auto covariance = math::linalg::Matrix<double>::identity(2);
math::linalg::vec first_mean{10.0, 20.0};
math::linalg::vec second_mean{90.0, 20.0};

auto s1 = math::linalg::Gaussian::create(
    first_mean, covariance, first_distribution);
auto s2 = math::linalg::Gaussian::create(
    second_mean, covariance, second_distribution);

if (s1 == math::solver::SolveStatus::Success &&
    s2 == math::solver::SolveStatus::Success) {
    math::filters::GaussianComponent a;
    a.distribution = first_distribution;
    a.weight = 0.6;

    math::filters::GaussianComponent b;
    b.distribution = second_distribution;
    b.weight = 0.4;

    math::filters::GaussianMixtureFilter filter;
    auto status = filter.initialize(std::vector<math::filters::GaussianComponent>{a, b});
}
```

After initialization, `predict(F, Q)` predicts the components and `update(measurement, H, R)` corrects them. `estimate()` returns a combined mean and covariance; `most_likely_component()` returns the highest-weight component; `components()` provides the component list. Updates can prune low-weight components and merge similar ones.

`GMMOptions` controls mixture management:

| Option | Meaning | Default |
|---|---|---:|
| `max_components` | Maximum number of components retained | `8` |
| `min_weight` | Weight threshold used to prune weak components | `0.01` |
| `merge_threshold` | Similarity threshold used when merging components | `2.0` |
| `covariance_floor` | Minimum variance floor used to stabilize covariance | `1e-6` |

Example: to retain up to twelve hypotheses, construct the filter with `GMMOptions` whose `max_components` is `12`. Larger mixtures can preserve more alternatives but require more computation. If pruning would remove every component, the filter keeps the strongest one.

## ParticleFilter (Monte Carlo localization)

`math::filters::mcl::ParticleFilter` represents a distribution using particles. Each particle contains a state vector and a weight. A motion callback advances each particle, and a log-likelihood callback scores how well its state explains a measurement. Resampling draws a new population with greater representation from higher-weight particles.

```cpp
math::random::Rng rng(1234); // fixed seed for repeatable testing

math::filters::mcl::ParticleFilterOptions options;
options.particle_count = 300;
options.resample_after_update = true;

math::filters::mcl::ParticleFilter filter(rng, options);

// Each sampled state must have three values, for example x, y, heading.
auto sampler = [](math::random::RandomSource& random,
                  math::linalg::vec& state) {
    state = math::linalg::vec{
        100.0 * random.uniform01(),
        100.0 * random.uniform01(),
        math::TAU * random.uniform01()
    };
    return math::solver::SolveStatus::Success;
};

auto status = filter.initialize_random(3, sampler);
```

Once initialized:

- `predict(control, motion_model)` advances each particle. The callback receives the particle state, control vector, random source, and output vector for the next state.
- `update(measurement, likelihood_model)` scores particles. The callback returns **log likelihood** through its output parameter.
- `normalize_weights()` normalizes particle weights.
- `resample()` draws a new particle population based on weights.
- `estimate()` returns a weighted mean, covariance, and effective sample size.
- `particles()` provides access to the particle list.

Example likelihood callback shape:

```cpp
math::filters::mcl::ParticleFilter::LogLikelihoodModel likelihood =
    [](const math::linalg::vec& state,
       const math::linalg::vec& measurement,
       double& log_likelihood) {
        // Replace with a sensor model. Higher log likelihood means
        // this state better explains the measurement.
        const double error = state(0) - measurement(0);
        log_likelihood = -0.5 * error * error;
        return math::solver::SolveStatus::Success;
    };
```

This likelihood is only a simple illustration; a real model should account for the sensor's units and noise distribution.

`GaussianMotionModel` is a helper for a state transition with Gaussian process noise. It can be useful when the model is naturally expressed as a transition function plus covariance.

`ParticleFilterOptions`:

| Option | Meaning | Default |
|---|---|---:|
| `particle_count` | Number of particles | `500` |
| `resample_ess_ratio` | Resampling threshold as a fraction of particle count | `0.5` |
| `min_weight` | Optional minimum weight floor | `0` (off) |
| `resample_after_update` | Automatically resample after updates | `true` |

The **effective sample size (ESS)** indicates how broadly particle weights contribute to the estimate. A low ESS means a small number of particles dominate. More particles may improve coverage but increase computation time.

## API reference pages

- [Kalman filter]({{ site.baseurl }}/reference/kalman-filter.html)
- [Extended Kalman filter]({{ site.baseurl }}/reference/extended-kalman-filter.html)
- [Gaussian mixture filter]({{ site.baseurl }}/reference/gaussian-mixture-filter.html)
- [Particle filter]({{ site.baseurl }}/reference/particle-filter.html)
