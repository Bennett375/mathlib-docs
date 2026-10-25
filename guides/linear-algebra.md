---
layout: default
title: Linear Algebra
parent: Guides
nav_order: 4
---

# Linear Algebra

## Matrix and Vector

`math::linalg::Matrix<T>` and `math::linalg::Vector<T>` provide the matrix and vector operations used throughout the library. They support `int`, `float`, and `double`; `math::linalg::vec` is shorthand for `Vector<double>`.

Matrices support element access, addition and subtraction, matrix multiplication, scalar operations, transpose, determinant, and inverse. Vectors support element access and vector operations including norms, distance, normalization, dot product, cross product, angle between vectors, and projection. Factory methods such as `identity()` and `zeros()` help construct common values.

```cpp
math::linalg::Matrix<double> A{
    {2.0, 1.0},
    {1.0, 3.0}
};
math::linalg::vec b{5.0, 7.0};

math::linalg::vec scaled = b * 2.0;
auto identity = math::linalg::Matrix<double>::identity(2);
auto zero_vector = math::linalg::vec::zeros(2);
```

Use a vector for a state or measurement and a matrix for transformations, covariance, or a system of equations. Operations require compatible dimensions.

## Gaussian distribution

`math::linalg::Gaussian` represents a multivariate Gaussian using a mean vector and covariance matrix. Create it through the factory; a default-constructed instance is invalid until creation succeeds.

```cpp
math::linalg::Gaussian belief;
math::linalg::vec mean{30.0, 45.0};
auto covariance = math::linalg::Matrix<double>::identity(2) * 4.0;

auto status = math::linalg::Gaussian::create(mean, covariance, belief);
if (status == math::solver::SolveStatus::Success) {
    math::linalg::vec candidate{31.0, 44.0};
    auto score = belief.log_pdf(candidate);
    if (score.success()) {
        double log_probability_density = score.value;
    }
}
```

- `mean()` and `covariance()` return the distribution parameters.
- `mahalanobis_distance()` and `mahalanobis_squared()` measure a point's distance from the mean relative to covariance.
- `pdf()` and `log_pdf()` evaluate probability density and log-density.
- These evaluation methods return a result containing both `status` and `value`; check `success()` before using the value.

## Solver

`math::solver::Solver<T>` solves linear systems of the form `A * x = b`. The result contains a status, solution vector, residual norm, and iteration count.

```cpp
math::linalg::Matrix<double> A{
    {3.0, 1.0},
    {1.0, 2.0}
};
math::linalg::vec b{9.0, 8.0};

auto result = math::solver::Solver<double>::solve(A, b);
if (result.success()) {
    math::linalg::vec x = result.solution;
    double residual = result.residual_norm;
}
```

Available methods:

| Method | Intended use | Notes |
|---|---|---|
| `Automatic` | General square systems | Uses LU by default |
| `GaussianElimination` | General square systems | Uses the same path as LU in this implementation |
| `LU` | General square systems | General-purpose direct method |
| `Cholesky` | Symmetric positive-definite systems | Often appropriate for valid covariance-related systems |
| `QR` | Least-squares and overdetermined systems | Requires at least as many rows as columns |
| `ConjugateGradient` | Iterative solution where applicable | Uses tolerance and iteration limit; can return `NoConvergence` |

For example, request QR explicitly instead of relying on the automatic choice:

```cpp
auto fit = math::solver::Solver<double>::solve(
    A, b, math::solver::SolveMethod::QR);
```

`SolveOptions<T>` controls numerical tolerance, pivot tolerance, maximum iterations, and finite-value checking. LU, Cholesky, and QR factorizations are also exposed directly for workflows that reuse a factorization for multiple right-hand sides.

The vector solve returns a `SolveResult<T>` and status. The matrix overload for multiple right-hand sides returns a matrix and throws if a column solve fails. Direct matrix/vector operations may also throw on misuse, including incompatible dimensions, out-of-range access, division by zero, or attempting to invert a singular matrix.

## API reference pages

- [Matrix, vector, and Gaussian API]({{ site.baseurl }}/reference/linear-algebra.html)
- [Solver API]({{ site.baseurl }}/reference/solver.html)
