---
layout: default
title: Task Recipes
parent: Guides
nav_order: 7
---

# Task Recipes

These examples show how the modules can be combined for common tasks. They omit robot-specific sensor and motor code; adapt the models, units, timing, and error handling to your robot.

## Hold a mechanism at a target

```cpp
math::control::PIDOptions options;
options.output_min = -1.0;
options.output_max = 1.0;
options.integral_limit = 0.25;
options.derivative_filter_alpha = 0.2;

math::control::PIDController controller(
    math::control::PIDGains(0.8, 0.05, 0.02), options);

double command = controller.update(target - measured, dt);
```

Update once per control cycle, use a consistent `dt`, and map the returned command to your actuator's safe range.

## Turn to a heading

```cpp
double error = math::angle_error(target_heading, current_heading);
double command = turn_controller.update(error, dt);
```

Using `angle_error` prevents a long turn when the target and current headings are on opposite sides of the wrap boundary.

## Fuse a distance sensor with a motion model

Given an initialized `filter` whose state is `{position, velocity}`:

{% raw %}
```cpp
const double dt = 0.02;

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

filter.predict(F, Q);
if (filter.update(math::linalg::vec{measured_distance}, H, R) ==
    math::solver::SolveStatus::Success) {
    math::linalg::vec estimate = filter.state();
}
```
{% endraw %}

The state is `{position, velocity}`. Tune `Q` for trust in the motion model and `R` for trust in the distance sensor.

## Estimate a nonlinear system

Use `ExtendedKalmanFilter` when the motion or measurement model is nonlinear. Supply the state function, state Jacobian, measurement function, measurement Jacobian, and optionally a residual function for wrapped angles. See the [EKF API example]({{ site.baseurl }}/reference/extended-kalman-filter.html) for a constant-acceleration model.

## Keep multiple position hypotheses

Use `GaussianMixtureFilter` when a measurement leaves more than one plausible state. Initialize one `GaussianComponent` per hypothesis, run `predict` and `update`, then inspect `components()` or `most_likely_component()`. The combined `estimate()` includes uncertainty between hypotheses.

## Localize with particles

Given a state sampler, a motion model, and a likelihood callback:

```cpp
math::random::Rng random(1234);
math::filters::mcl::ParticleFilter filter(random);

filter.initialize_random(state_dimension, sampler);

filter.predict(control, motion_model);
filter.update(measurement, likelihood_model);

math::filters::mcl::ParticleEstimate estimate = filter.estimate();
if (estimate.success()) {
    math::linalg::vec mean = estimate.mean;
    double ess = estimate.effective_sample_size;
}
```

Use a log-likelihood callback, include process noise in the motion model, and monitor ESS before changing the particle count.

## Trilaterate from two distances

```cpp
math::Circle circle_a(anchor_a, distance_a);
math::Circle circle_b(anchor_b, distance_b);

std::vector<math::Point> candidates =
    math::circle_intersections(circle_a, circle_b);
```

The result can contain zero, one, or two points. Use another sensor, a heading estimate, or field constraints to choose between candidates.

## Fit a line or solve a least-squares problem

```cpp
math::linalg::Matrix<double> design{
    {1.0, 0.0},
    {1.0, 1.0},
    {1.0, 2.0}
};
math::linalg::vec observations{1.0, 2.0, 3.0};

auto fit = math::solver::Solver<double>::solve(
    design, observations, math::solver::SolveMethod::QR);

if (fit.success()) {
    math::linalg::vec coefficients = fit.solution;
}
```

QR is the appropriate choice when the system has more equations than unknowns and the design matrix has full column rank.

## Score a candidate state

Given a valid Gaussian `belief` and a candidate vector:

```cpp
auto score = belief.log_pdf(candidate);
if (score.success()) {
    double log_density = score.value;
}
```

Use `log_pdf` rather than `pdf` when comparing very small likelihoods or when the density could underflow.

## Create repeatable simulation noise

```cpp
math::random::Rng random(1234);
double sensor_noise = random.normal(0.0, 0.1);
```

A nonzero seed gives a repeatable sequence on the same platform. Use a custom `RandomSource` when you need recorded noise or a deterministic test double.

## Log a state or geometry value

Given a `double` vector, a `Point`, and a `Circle`:

```cpp
std::string state_text = state.as<std::string>();
std::string point_text = point.as<std::string>();
std::string circle_text = circle.as<std::string>();
```

The string conversions are intended for debugging and logs. Do not use them as a stable serialization format.

## Related guides

- [PID control]({{ site.baseurl }}/guides/control.html)
- [State estimation]({{ site.baseurl }}/guides/state-estimation.html)
- [Field geometry]({{ site.baseurl }}/guides/geometry.html)
- [Linear algebra]({{ site.baseurl }}/guides/linear-algebra.html)
