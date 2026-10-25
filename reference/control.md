---
layout: default
title: PID Controller
parent: References
nav_order: 3
---

# PID Controller

`math::control::PIDController` calculates an output from a control error and elapsed time. For a task-oriented introduction, see the [PID control guide]({{ site.baseurl }}/guides/control.html).

## `PIDGains`

| Field | Type | Default | Meaning |
|---|---|---:|---|
| `kp` | `double` | `0.0` | Proportional gain |
| `ki` | `double` | `0.0` | Integral gain |
| `kd` | `double` | `0.0` | Derivative gain |

Constructor: `PIDGains(double p = 0.0, double i = 0.0, double d = 0.0)`.

## `PIDOptions`

| Field | Type | Default | Meaning |
|---|---|---:|---|
| `integral_limit` | `double` | `0.0` | Positive value clamps the integral to `[-limit, +limit]`; zero or negative disables the clamp |
| `output_min` | `double` | `-std::numeric_limits<double>::max()` | Lower output clamp; effectively unbounded by default |
| `output_max` | `double` | `std::numeric_limits<double>::max()` | Upper output clamp; effectively unbounded by default |
| `derivative_filter_alpha` | `double` | `1.0` | Blend factor; `1.0` means no filtering; values outside `[0, 1]` are clamped when used |

Set `output_min <= output_max`. The controller does not validate the ordering.

## `PIDState`

| Field | Type | Default | Meaning |
|---|---|---:|---|
| `integral` | `double` | `0.0` | Accumulated integral |
| `previous_error` | `double` | `0.0` | Error from the previous valid update |
| `filtered_derivative` | `double` | `0.0` | Derivative after filtering |
| `initialized` | `bool` | `false` | Whether at least one valid update has run |

## `PIDController`

| Method | Returns | Behavior |
|---|---|---|
| `PIDController(const PIDGains& gains, const PIDOptions& options = PIDOptions())` | Controller | Stores settings; does not validate gain or limit ranges |
| `double update(double error, double dt)` | Controller output | Computes one control cycle; `dt` is in seconds and must be finite and greater than zero |
| `void reset()` | Nothing | Clears integral, previous error, filtered derivative, initialized flag, and last output |
| `const PIDGains& gains() const` | Current gains | Read-only access |
| `void set_gains(const PIDGains& gains)` | Nothing | Changes gains without clearing controller history |
| `const PIDState& state() const` | Current state | Read-only access; the contents change on the next update |
| `double last_output() const` | Last output | Returns `0.0` before the first valid update |

**Update behavior**

1. Invalid samples are ignored. If `error` or `dt` is not finite, or `dt <= 0`, `update` returns `last_output()` and leaves the state unchanged.
2. The derivative is calculated from error, not from the measurement. On the first sample the raw derivative is treated as zero to avoid a derivative kick. With `alpha < 1`, the first filtered value is `(1 - alpha) * filtered_derivative`, which is normally zero.
3. On later samples, `raw = (error - previous_error) / dt`. The filtered derivative is `alpha * raw + (1 - alpha) * previous_filtered`.
4. The integral candidate is `previous_integral + error * dt`. If `integral_limit > 0`, it is clamped to `[-integral_limit, integral_limit]`.
5. The output is `kp * error + ki * integral + kd * derivative`.
6. Conditional anti-windup: if the output is above `output_max` while the error is positive, or below `output_min` while the error is negative, the integral is reverted to its previous value and the output is recomputed.
7. The output is clamped to `[output_min, output_max]`, then the state and last output are updated.

`set_gains()` does not reset the integral or derivative history. Call `reset()` when switching control modes, changing the meaning of the error, or restarting a routine.

There is no feedforward term, no derivative-on-measurement option, no automatic tuning, and no gain scheduling. Those behaviors belong in your robot code.

```cpp
math::control::PIDOptions options;
options.output_min = -1.0;
options.output_max = 1.0;
options.integral_limit = 0.25;
options.derivative_filter_alpha = 0.2;

math::control::PIDController controller(
    math::control::PIDGains(0.8, 0.05, 0.02), options);

double command = controller.update(target - measured, 0.02);
controller.set_gains(math::control::PIDGains(1.0, 0.05, 0.03));

if (controller.state().initialized) {
    double accumulated_error = controller.state().integral;
}
```
