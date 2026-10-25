---
layout: default
title: Solvers
parent: References
nav_order: 9
---

# Solvers

The solver templates are explicitly instantiated for `int`, `float`, and `double`. Use `double` for numerical work. Integer arithmetic is not suitable for numerically sensitive solving, and converting a result back to `int` truncates fractional values. For the shared status values, see [Status codes]({{ site.baseurl }}/reference/conventions.html#status-codes).

## Enumerations

`SolveMethod`:

| Value | Meaning |
|---|---|
| `Automatic` | Selects LU in this implementation |
| `GaussianElimination` | Uses the same LU path as `LU` |
| `LU` | General square systems with partial pivoting |
| `Cholesky` | Symmetric positive-definite systems |
| `QR` | Least-squares and overdetermined systems |
| `ConjugateGradient` | Iterative square-system solve |

`SolveStatus` is the shared status enum described in [status codes]({{ site.baseurl }}/reference/conventions.html#status-codes).

## `SolveOptions<T>`

| Field | Default | Meaning |
|---|---:|---|
| `tolerance` | `1e-6` | Iterative convergence tolerance and part of the Cholesky symmetry tolerance |
| `pivot_tolerance` | `1e-7` | Relative pivot threshold for singularity, positive-definiteness, and QR rank decisions |
| `max_iterations` | `100` | Maximum conjugate-gradient iterations |
| `check_finite` | `true` | Reject NaN and infinity before solving |

Tolerances should be nonnegative. With `T = int`, the default tolerance and pivot tolerance truncate to zero; use a floating-point type for numerical sensitivity. Setting `check_finite = false` can save work when inputs are already known to be finite, but it also allows invalid values to propagate.

## Result types

`SolveResult<T>`:

| Field or method | Type | Meaning |
|---|---|---|
| `status` | `SolveStatus` | Defaults to `NumericalFailure` |
| `solution` | `linalg::Vector<T>` | Solution vector; empty by default |
| `residual_norm` | `double` | Residual norm when available |
| `iterations` | `std::size_t` | Iteration count; direct solves report `1` on success, conjugate gradient reports its actual count |
| `success()` | `bool` | True when `status == Success` |

`LUResult<T>`:

| Field or method | Type | Meaning |
|---|---|---|
| `status` | `SolveStatus` | Decomposition status |
| `L` | `linalg::Matrix<T>` | Unit lower-triangular factor |
| `U` | `linalg::Matrix<T>` | Upper-triangular factor |
| `permutation` | `std::vector<std::size_t>` | `permutation[i]` is the original row occupying row `i` of `P*A` |
| `permutation_sign` | `int` | `+1` or `-1` for the row permutation |
| `success()` | `bool` | True when `status == Success` |

`CholeskyResult<T>`:

| Field or method | Type | Meaning |
|---|---|---|
| `status` | `SolveStatus` | Decomposition status |
| `L` | `linalg::Matrix<T>` | Lower-triangular factor with `A = L*L^T` |
| `success()` | `bool` | True when `status == Success` |

`QRResult<T>`:

| Field or method | Type | Meaning |
|---|---|---|
| `status` | `SolveStatus` | Decomposition status |
| `Q` | `linalg::Matrix<T>` | `m x n` factor; columns are orthonormal when full column rank |
| `R` | `linalg::Matrix<T>` | `n x n` upper-triangular factor |
| `rank` | `std::size_t` | Numerical rank estimate |
| `success()` | `bool` | True when `status == Success` |

`LUResult`, `CholeskyResult`, and `QRResult` default to `NumericalFailure` with empty factors. `LUResult::permutation_sign` starts at `+1`, and `QRResult::rank` starts at `0`.

## Free functions

All free functions except `residual_norm` accept a final `const SolveOptions<T>&` argument that defaults to `SolveOptions<T>()`.

| Function | Returns | Requirements and behavior |
|---|---|---|
| `lu_decompose(A, options)` | `LUResult<T>` | Square matrix; partial pivoting; singular pivots return `SingularMatrix` |
| `cholesky_decompose(A, options)` | `CholeskyResult<T>` | Square, symmetric, positive definite; non-symmetry beyond tolerance or non-positive-definite input returns `NotPositiveDefinite` |
| `qr_decompose(A, options)` | `QRResult<T>` | `rows >= cols`; modified Gram-Schmidt with reorthogonalization; rank-deficient columns leave zero columns in `Q` but the decomposition itself can still succeed |
| `solve_lu(decomposition, b, options)` | `SolveResult<T>` | Uses existing LU factors; checks dimensions and finite values |
| `solve_cholesky(decomposition, b, options)` | `SolveResult<T>` | Uses existing Cholesky factors |
| `solve_qr(decomposition, b, options)` | `SolveResult<T>` | Requires full column rank; otherwise returns `SingularMatrix` |
| `determinant(A, options)` | `T` | Requires a square matrix; returns `T()` if LU fails; throws `std::invalid_argument` for non-square input |
| `numerical_rank(A, options)` | `std::size_t` | Uses QR; returns `0` if QR fails |
| `residual_norm(A, x, b)` | `double` | Computes the 2-norm of `A*x - b`; throws `std::invalid_argument` on incompatible dimensions |

## `Solver<T>`

| Method | Returns | Behavior |
|---|---|---|
| `static SolveResult<T> solve(const linalg::Matrix<T>& A, const linalg::Vector<T>& b, SolveMethod method = Automatic, const SolveOptions<T>& options = SolveOptions<T>())` | `SolveResult<T>` | Solves `A*x = b` |
| `static linalg::Matrix<T> solve(const linalg::Matrix<T>& A, const linalg::Matrix<T>& B, SolveMethod method = Automatic, const SolveOptions<T>& options = SolveOptions<T>())` | `linalg::Matrix<T>` | Solves `A*X = B` column by column |

**Method requirements and results**

| Method | Matrix shape | Notes |
|---|---|---|
| `Automatic` | Same as LU | Always selects LU in this implementation |
| `GaussianElimination` | Square | Uses the LU path |
| `LU` | Square | Partial pivoting; singular systems return `SingularMatrix` |
| `Cholesky` | Square, symmetric positive definite | Returns `NotPositiveDefinite` when the matrix is not SPD |
| `QR` | `rows >= cols` | Solves least squares when the system is overdetermined and full column rank; rank deficiency returns `SingularMatrix` |
| `ConjugateGradient` | Square | Intended for symmetric positive-definite systems; a nonpositive or non-finite search denominator returns `NumericalFailure` |

**Behavior and failure notes**

- All methods convert inputs to double precision internally. Results are converted back to `T`; integer solutions truncate fractional values. Use `double` for numerical work.
- `Solver<T>::solve(A, b)` checks `A.rows() == b.size()` and returns `InvalidDimensions` if they differ. It checks finite values when `check_finite` is true.
- `Solver<T>::solve` sets `residual_norm` on success for the direct methods. Conjugate gradient also reports its final residual. The lower-level `solve_lu`, `solve_cholesky`, and `solve_qr` functions leave `residual_norm` at its default value.
- Conjugate gradient does not verify symmetry or positive definiteness; it assumes an SPD system. It returns `Success` with `iterations = 0` when the initial residual is already within `tolerance`. If it reaches `max_iterations`, it returns `NoConvergence` with the last iterate and residual.
- `solve_qr` requires `decomposition.rank == number_of_columns`; a rank-deficient least-squares problem returns `SingularMatrix` rather than a minimum-norm solution.
- `qr_decompose` can succeed with `rank < cols`. Inspect `rank` if you need to detect a rank-deficient design matrix.
- The matrix right-hand-side overload throws `std::invalid_argument` if `A.rows() != B.rows()`. It throws `std::runtime_error` if any column solve fails; use the vector overload if you need per-column statuses.
- `determinant` and `numerical_rank` are convenience wrappers. They return a fallback value on failure; use `lu_decompose` or `qr_decompose` when you need the exact reason.
- `math::solver::detail` is internal and may change without notice.

```cpp
math::linalg::Matrix<double> A{
    {4.0, 1.0},
    {1.0, 3.0}
};
math::linalg::vec b{1.0, 2.0};

auto result = math::solver::Solver<double>::solve(A, b);
if (result.success()) {
    math::linalg::vec x = result.solution;
    double residual = result.residual_norm;
}
```

Reusing a factorization for multiple right-hand sides:

```cpp
auto lu = math::solver::lu_decompose(A);
if (lu.success()) {
    auto x1 = math::solver::solve_lu(lu, b);
    auto x2 = math::solver::solve_lu(lu, another_b);
}
```

Least-squares fit with QR:

```cpp
math::linalg::Matrix<double> design{
    {1.0, 0.0},
    {1.0, 1.0},
    {1.0, 2.0}
};
math::linalg::vec observations{1.0, 2.0, 3.0};

auto fit = math::solver::Solver<double>::solve(
    design, observations, math::solver::SolveMethod::QR);
```
