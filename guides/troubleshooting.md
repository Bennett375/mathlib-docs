---
layout: default
title: Troubleshooting
parent: Guides
nav_order: 8
---

# Troubleshooting

| Symptom | Likely cause | What to check |
|---|---|---|
| `InvalidDimensions` on filter initialization | State and covariance sizes differ, or the distribution is invalid | Check `state.size()`, `covariance.rows()`, and `covariance.cols()` |
| `InvalidDimensions` during predict/update | `F`, `Q`, `B`, `u`, `H`, or `R` shape is wrong | Write the expected shapes next to the model and compare |
| `NotPositiveDefinite` | A Cholesky or Gaussian covariance check failed | Symmetrize the covariance and add a small positive diagonal floor |
| `NonFiniteInput` | Sensor value, control, matrix, or callback output is NaN or infinity | Log the raw input before it reaches MathLib |
| `SingularMatrix` | System is singular or a least-squares design matrix is rank deficient | Check the model, remove redundant equations, or regularize |
| `NoConvergence` | Conjugate gradient hit `max_iterations` or is poorly scaled | Increase iterations, loosen tolerance, rescale, or use a direct method |
| `NumericalFailure` | Ill-conditioning, an invalid intermediate result, or a failed post-check | Inspect values near zero, scale the problem, and validate the model |
| `UnsupportedMethod` | Required callback is empty or the selected method is unsupported | Check that the `std::function` is assigned and the method is valid for the shape |
| PID output stays at a limit | Output saturation, integral windup, or the wrong error sign | Inspect `last_output()`, `state().integral`, and the output limits |
| PID oscillates | Excessive `kp` or `kd`, noisy derivative, or inconsistent `dt` | Lower gains, filter the derivative, and use a steady control period |
| Particle filter collapses onto one state | Likelihood is too sharp, particles are too few, or motion noise is too small | Log ESS and weights; increase diversity or soften the likelihood |
| GMM loses a hypothesis | `min_weight` is too high, `merge_threshold` is too high, or `max_components` is too low | Inspect component weights before and after `update` |
| `.as<T>()` fails to compile | No conversion specialization exists for that source/target pair | Use one of the built-in conversions or add a `math::conversion` specialization |
| `std::out_of_range` | Matrix or vector index is outside its dimensions | Check loop bounds and `rows()`/`cols()`/`size()` |
| `std::domain_error` | Singular inverse, zero-vector normalization, zero-vector angle, or zero-vector projection | Validate the matrix or vector before the operation |
| `std::invalid_argument` | Dimension mismatch, division by zero, non-square determinant/inverse, or unequal initializer rows | Check dimensions and scalar divisors |
| Link error mentioning `mathlib` | Headers were found but the archive was not linked | Add `bin/mathlib.a` or compile the `src/mathlib` sources |
| Build cannot find `mathlib/mathlib.hpp` | Include path does not contain the repository's `include/` directory | Add `include/` to the compiler include path |

For status values and their meaning, see [API conventions and status codes]({{ site.baseurl }}/reference/conventions.html#status-codes). For unsupported features and API constraints, see [Limitations]({{ site.baseurl }}/development/limitations.html).
