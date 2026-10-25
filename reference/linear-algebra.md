---
layout: default
title: Linear Algebra
parent: References
nav_order: 8
---

# Matrix, Vector, and Gaussian

`math::linalg::Matrix<T>` and `math::linalg::Vector<T>` provide the matrix and vector operations used throughout the library. `math::linalg::Gaussian` represents a multivariate normal distribution. For a task-oriented introduction, see the [linear algebra guide]({{ site.baseurl }}/guides/linear-algebra.html).

## `Matrix<T>`

`Matrix<T>` is a dense, row-major matrix that owns a `std::vector<T>`. It is a value type and can be copied or moved.

### Matrix constructors

| Constructor | Result |
|---|---|
| `Matrix()` | `0 x 0` empty matrix |
| `Matrix(std::size_t rows, std::size_t cols)` | `rows x cols` matrix with value-initialized elements (zero for arithmetic types) |
| `Matrix(std::size_t rows, std::size_t cols, const T& initial_value)` | Matrix filled with `initial_value` |
| `Matrix(std::initializer_list<std::initializer_list<T>> values)` | Matrix whose rows are the nested lists; every row must have the same length |

An empty initializer list produces a `0 x 0` matrix. Unequal row lengths throw `std::invalid_argument`.

### Dimensions and element access

| Member | Returns | Behavior |
|---|---|---|
| `std::size_t rows() const` | Row count | |
| `std::size_t cols() const` | Column count | |
| `bool empty() const` | Flag | True when either dimension is zero |
| `bool is_square() const` | Flag | True when `rows() == cols()` |
| `T& operator()(std::size_t row, std::size_t col)` | Element reference | Bounds-checked; throws `std::out_of_range` |
| `const T& operator()(std::size_t row, std::size_t col) const` | Const element reference | Bounds-checked; throws `std::out_of_range` |
| `T& at(std::size_t row, std::size_t col)` | Element reference | Same bounds checking as `operator()` |
| `const T& at(std::size_t row, std::size_t col) const` | Const element reference | Same bounds checking as `operator()` |
| `const std::vector<T>& data() const` | Underlying storage | Row-major order; read-only |
| `void fill(const T& value)` | Nothing | Assigns `value` to every element |

Both `operator()` and `at()` are checked. The underlying `data()` vector is exposed only as a const reference.

### Arithmetic

| Operator | Requirement | Failure |
|---|---|---|
| `Matrix<T> operator+(const Matrix<T>& other) const` | Same dimensions | `std::invalid_argument` on mismatch |
| `Matrix<T> operator-(const Matrix<T>& other) const` | Same dimensions | `std::invalid_argument` on mismatch |
| `Matrix<T> operator*(const Matrix<T>& other) const` | `this->cols() == other.rows()` | `std::invalid_argument` on mismatch |
| `Vector<T> operator*(const Vector<T>& other) const` | `this->cols() == other.size()` | `std::invalid_argument` on mismatch |
| `Matrix<T> operator*(const T& scalar) const` | None | Arithmetic overflow is the caller's responsibility |
| `Matrix<T> operator/(const T& scalar) const` | Scalar is not zero | `std::invalid_argument` on zero |
| `Matrix<T>& operator+=(const Matrix<T>& other)` | Same dimensions | `std::invalid_argument` on mismatch |
| `Matrix<T>& operator-=(const Matrix<T>& other)` | Same dimensions | `std::invalid_argument` on mismatch |
| `Matrix<T>& operator*=(const T& scalar)` | None | Arithmetic overflow is the caller's responsibility |
| `Matrix<T> operator*(const T& scalar, const Matrix<T>& matrix)` | None | Scalar-left convenience overload |

### Linear algebra and factories

| Member | Returns | Behavior |
|---|---|---|
| `Matrix<T> transpose() const` | Transposed matrix | Swaps rows and columns |
| `T determinant() const` | Determinant | Requires a square matrix; returns `T(1)` for `0 x 0` and `T()` for a singular matrix; throws `std::invalid_argument` when non-square |
| `Matrix<T> inverse() const` | Inverse | Requires a square matrix; returns an empty matrix for `0 x 0`; throws `std::domain_error` for a singular or numerically singular matrix; throws `std::invalid_argument` when non-square |
| `static Matrix<T> identity(std::size_t size)` | Identity matrix | `size x size` with ones on the diagonal |
| `static Matrix<T> zeros(std::size_t rows, std::size_t cols)` | Zero matrix | All elements set to `T()` |

`determinant()` and `inverse()` use a wider internal type and partial pivoting. They are convenience operations, not the preferred path for solving systems. Prefer `math::solver` factorizations, especially for covariance matrices and repeated solves.

Only `Matrix<int>`, `Matrix<float>`, and `Matrix<double>` are explicitly instantiated. Use those types unless you are prepared to provide your own template definitions. Conversion to `std::string` exists only for `Matrix<double>`.

## `Vector<T>`

`Vector<T>` inherits from `Matrix<T>` and represents a column vector with `size()` rows and one column. The inherited `rows()` method equals `size()`, and `cols()` is one.

### Vector constructors

| Constructor | Result |
|---|---|
| `explicit Vector(std::size_t size)` | Zero-filled vector of length `size` |
| `Vector(std::size_t size, const T& initial_value)` | Vector filled with `initial_value` |
| `Vector(std::initializer_list<T> values)` | Vector containing the listed values |

### Elements and arithmetic

| Member | Returns | Behavior |
|---|---|---|
| `std::size_t size() const` | Length | Number of rows |
| `bool empty() const` | Flag | True when the length is zero |
| `T& operator()(std::size_t row)` | Element reference | Bounds-checked; throws `std::out_of_range` |
| `const T& operator()(std::size_t row) const` | Const element reference | Bounds-checked; throws `std::out_of_range` |
| `Vector operator+(const Vector& other) const` | Sum | Same size required; throws `std::invalid_argument` on mismatch |
| `Vector operator-(const Vector& other) const` | Difference | Same size required; throws `std::invalid_argument` on mismatch |
| `Vector operator-() const` | Negation | Element-wise negation |
| `Vector operator*(const T& scalar) const` | Scaled vector | |
| `Vector operator/(const T& scalar) const` | Divided vector | Throws `std::invalid_argument` for zero |
| `Vector& operator+=(const Vector& other)` | This vector | Same size required; throws `std::invalid_argument` on mismatch |
| `Vector& operator-=(const Vector& other)` | This vector | Same size required; throws `std::invalid_argument` on mismatch |
| `Vector& operator*=(const T& scalar)` | This vector | |

### Vector operations

| Member or free function | Returns | Behavior |
|---|---|---|
| `double norm() const` | Euclidean norm | Uses a double accumulator |
| `double squared_norm() const` | Squared norm | Uses a double accumulator |
| `double distance_to(const Vector& other) const` | Euclidean distance | Same size required; throws `std::invalid_argument` on mismatch |
| `Vector<double> normalized() const` | Unit vector | Returns a double vector; throws `std::domain_error` for a zero vector |
| `double angle_between(const Vector& other) const` | Angle in `[0, pi]` radians | Same size required; throws `std::invalid_argument` on mismatch and `std::domain_error` if either vector is zero |
| `Vector<double> projection_onto(const Vector& other) const` | Projection of this vector onto `other` | Same size required; throws `std::invalid_argument` on mismatch and `std::domain_error` when `other` is zero |
| `Matrix<T> mat_from_diag() const` | Square diagonal matrix | Places this vector on the diagonal |
| `static Vector zeros(std::size_t size)` | Zero vector | |
| `double dot(const Vector<T>& a, const Vector<T>& b)` | Dot product | Same size required; throws `std::invalid_argument` on mismatch; returns `double` |
| `Vector<T> cross(const Vector<T>& a, const Vector<T>& b)` | Cross product | Both vectors must have exactly three elements; otherwise throws `std::invalid_argument` |
| `Vector<T> operator*(const T& scalar, const Vector<T>& vector)` | Scaled vector | Scalar-left convenience overload |

`math::linalg::vec` is a typedef for `Vector<double>`. Only `Vector<int>`, `Vector<float>`, and `Vector<double>` are explicitly instantiated. Conversion to `std::string` exists only for `Vector<double>`.

Because `Vector<T>` inherits publicly from `Matrix<T>`, base members such as `at`, `data`, `fill`, `transpose`, `determinant`, `inverse`, and `identity` are also visible. Matrix-specific operations generally reject a non-square vector shape, so prefer the vector-specific API above unless you deliberately need the matrix result.

```cpp
math::linalg::vec a{1.0, 2.0, 3.0};
math::linalg::vec b{4.0, 5.0, 6.0};

double d = math::linalg::dot(a, b);
math::linalg::vec c = math::linalg::cross(a, b);
math::linalg::vec unit = a.normalized();
math::linalg::Matrix<double> diagonal = a.mat_from_diag();
```

## `Gaussian`

`Gaussian` represents a multivariate normal distribution with a mean vector and a covariance matrix. A default-constructed Gaussian is invalid until `create` succeeds.

| Member | Returns | Behavior |
|---|---|---|
| `Gaussian()` | Invalid distribution | Dimension `0`, empty mean, empty covariance |
| `static solver::SolveStatus create(const Vector<double>& mu, const Matrix<double>& sigma, Gaussian& out)` | `SolveStatus` | Validates and factorizes the distribution |
| `bool valid() const` | Flag | Whether the distribution is usable |
| `std::size_t dim() const` | Dimension | Mean length |
| `const Vector<double>& mean() const` | Mean | Read-only reference |
| `const Matrix<double>& covariance() const` | Covariance | Read-only reference |
| `GaussianValueResult mahalanobis_squared(const Vector<double>& x) const` | Squared Mahalanobis distance | Invalid distribution returns `NumericalFailure`; wrong size returns `InvalidDimensions` |
| `GaussianValueResult mahalanobis_distance(const Vector<double>& x) const` | Mahalanobis distance | Square root of the squared value |
| `GaussianValueResult log_pdf(const Vector<double>& x) const` | Log probability density | Uses the factorized covariance |
| `GaussianValueResult pdf(const Vector<double>& x) const` | Probability density | `exp(log_pdf)`; may underflow to `0.0` or overflow to infinity with a successful status |

`GaussianValueResult`:

| Field or method | Type | Meaning |
|---|---|---|
| `status` | `math::solver::SolveStatus` | Result status; defaults to `NumericalFailure` |
| `value` | `double` | Numeric result; defaults to `0.0` |
| `success()` | `bool` | True when `status == Success` |

The default constructor sets `NumericalFailure` and `0.0`; `GaussianValueResult(status, value)` sets both fields explicitly.

`create` requirements:

- `mu` must be non-empty.
- `sigma` must be `n x n`.
- The covariance must contain only finite values.
- The covariance must be symmetric within the Cholesky solver's tolerance.
- The covariance must be positive definite, not merely positive semidefinite. Use a small positive variance floor instead of an exact zero variance.
- The mean is copied as given and is not checked for finiteness during creation. Supply finite mean values.

If creation fails, `out` is left invalid; check the returned status. Calling `valid()` is safe but returns false.

```cpp
math::linalg::Gaussian belief;
math::linalg::vec mean{30.0, 45.0};
math::linalg::Matrix<double> covariance =
    math::linalg::Matrix<double>::identity(2) * 4.0;

auto status = math::linalg::Gaussian::create(mean, covariance, belief);
if (status == math::solver::SolveStatus::Success) {
    auto score = belief.log_pdf(math::linalg::vec{31.0, 44.0});
    if (score.success()) {
        double log_density = score.value;
    }
}
```
