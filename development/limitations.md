---
layout: default
title: Limitations
parent: Development Summary
nav_order: 5
---

# Limitations

MathLib is intentionally focused on numerical building blocks. The sections below describe what it does not provide and the constraints that follow from its design.

## What MathLib does not provide

MathLib is a math library, not a complete robotics framework. It does not:

- Read VEX sensors or command motors.
- Provide a drivetrain or autonomous framework.
- Plan paths or generate trajectories.
- Automatically tune controllers or models.
- Include a logger or plotter.

Your robot program supplies the hardware interface, sensor data, system models, control loop timing, and any logging or visualization.

## Additional API constraints

- The public API is C++11 or newer.
- Matrices and vectors are dense; there is no sparse representation.
- Matrices cannot be resized in place; construct a new matrix when the shape changes.
- `Matrix<T>` and `Vector<T>` are explicitly instantiated only for `int`, `float`, and `double`.
- `math::solver` function templates and `Solver<T>` are explicitly instantiated only for those same three types.
- High-level filters and `Gaussian` use `double`.
- There is no automatic differentiation and no built-in numerical Jacobian for the EKF.
- The GMM has no nonlinear prediction model and no `B`/`u` control overload.
- There is no structured serialization, plotting, logging backend, or hardware abstraction.
- There is no unit checking; the caller must keep units consistent.
- The library does not provide real-time scheduling guarantees. Measure filter and solver cost on the target.
- `math::solver::detail` and private members are implementation details, not stable API.
