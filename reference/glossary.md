---
layout: default
title: Glossary
parent: References
nav_order: 12
---

# Glossary

| Term | Meaning | Example |
|---|---|---|
| **State** | Values describing a system at a given time | `{x, y, heading, velocity}` |
| **Covariance** | Matrix describing uncertainty and correlations in a state estimate | A larger position variance means less certainty about position |
| **Process noise (`Q`)** | Uncertainty assigned to the motion model | Increase it if wheel slip makes odometry less reliable |
| **Measurement noise (`R`)** | Uncertainty assigned to sensor measurements | Use a larger variance for a noisy distance sensor |
| **Jacobian** | Matrix of partial derivatives used to linearize a nonlinear function | Derivatives of the robot motion model with respect to state |
| **Effective sample size (ESS)** | Summary of how many particles meaningfully contribute | Low ESS means a few particles dominate the weights |
| **SPD** | Symmetric positive definite matrix | A covariance matrix suitable for Cholesky factorization |
| **Radians** | Angular unit used by MathLib's geometry and angle helpers | `math::PI` is half a turn |
