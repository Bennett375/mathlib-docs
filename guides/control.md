---
layout: default
title: PID Control
parent: Guides
nav_order: 1
---

# PID Control

`math::control::PIDController` calculates an output from a control error and elapsed time. It is useful for position, velocity, and heading loops.

The controller combines three terms:

- **Proportional (`kp`)** responds to the current error.
- **Integral (`ki`)** accumulates error over time and can reduce persistent steady-state error.
- **Derivative (`kd`)** responds to the rate of change of error and can help reduce overshoot.

In brief, P responds to current error, I accumulates past error, and D responds to the error's rate of change.

## Basic example

```cpp
math::control::PIDController flywheel(
    math::control::PIDGains(0.8, 0.02, 0.01));

double error = target_rpm - measured_rpm;
double output = flywheel.update(error, 0.02);
```

Call `update(error, dt)` once per control cycle. `error` and the gains must use units that make sense for your mechanism; `dt` is in seconds. The returned value is a numeric output, so convert or clamp it as required by your motor-command interface.

## Options and state

`PIDOptions` configures output limits, integral limiting, and derivative filtering.

```cpp
math::control::PIDOptions options;
options.output_min = -100.0;
options.output_max = 100.0;
options.integral_limit = 20.0;
options.derivative_filter_alpha = 0.25;

math::control::PIDController arm(
    math::control::PIDGains(1.0, 0.0, 0.05), options);
```

- `output_min` and `output_max` clamp the controller output. The defaults are effectively unbounded.
- `integral_limit` limits the accumulated integral. A value of `0` disables this limit.
- `derivative_filter_alpha` controls derivative filtering. The default `1.0` means no filtering; a value below `1.0` smooths the derivative response.
- `set_gains()` changes gains without reconstructing the controller.
- `reset()` clears controller history, which is useful when switching modes or restarting a control routine.
- `state()` exposes the current PID state; `last_output()` returns the most recent output.

Gains and options default to zero gains and neutral protection settings. Choose output and integral limits that are appropriate for the mechanism.

## Safeguards

The controller ignores invalid or non-finite samples and returns the previous output. It skips the first derivative calculation to avoid a derivative kick, clamps the output to configured limits, and limits integral accumulation when configured. These safeguards do not replace testing with the real mechanism.

## Next steps

- [PID controller API reference]({{ site.baseurl }}/reference/control.html)
- [Tuning guide]({{ site.baseurl }}/guides/tuning.html)
