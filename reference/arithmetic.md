---
layout: default
title: Arithmetic and Conversion
parent: References
nav_order: 2
---

# Constants, Arithmetic, and Conversion

The `math` namespace includes constants and helpers for common calculations, plus the `MathObj` conversion base class.

## Constants

| Symbol | Type | Value |
|---|---|---|
| `math::PI` | `double` | Pi |
| `math::TAU` | `double` | `2 * PI` |
| `math::e` | `double` | Euler's number |

## Arithmetic functions

| Signature | Returns | Behavior |
|---|---|---|
| `double mean(const std::vector<double>& values)` | Arithmetic mean | Returns `0.0` for an empty vector |
| `double mean(double first, double second)` | Arithmetic mean | Equivalent to `(first + second) / 2.0` |
| `int sign(int value)` | `-1`, `0`, or `1` | Compares the value with zero |
| `double sign(double value)` | `-1.0`, `0.0`, or `1.0` | A NaN value produces `0.0` because both comparisons are false |
| `double clamp(double value, double minimum = -100.0, double maximum = 100.0)` | Clamped value | Uses `max(minimum, min(maximum, value))`; does not validate that `minimum <= maximum` |
| `double power(double base, double exponent)` | `base` raised to `exponent` | Forwards to the standard power function; no domain checking |
| `double exp(double exponent)` | Natural exponential | Implemented through `power` |
| `double sqr(double value)` | `value * value` | No overflow or domain checking |
| `double sqrt(double value)` | Square root | Negative inputs produce NaN according to the standard library |
| `template <typename T> std::vector<std::vector<T>> make_groups(const std::vector<T>& values, std::size_t group_size, bool disjoint = true)` | Groups | A group size of zero returns no groups; a short final group is allowed |
| `template <typename T> std::vector<std::vector<T>> make_pairs(const std::vector<T>& values, bool disjoint = true)` | Pairs | Calls `make_groups(values, 2, disjoint)` |

For `make_groups`, `disjoint=true` advances by `group_size`. With `disjoint=false`, the window advances by one element, so groups can overlap by more than one element when `group_size` is greater than two.

Qualify `math::sqrt` if you also use `std::sqrt` or a C math `sqrt` in the same scope. The functions are thin wrappers; they do not add validation beyond the underlying operation.

## `MathObj` and conversions

`MathObj<Derived>` is a small base class that provides:

```cpp
template <typename T>
T as() const;
```

The conversion is implemented by `math::conversion<Object, Target>`. The primary template is intentionally undefined, so unsupported conversions fail at compile time rather than at runtime.

Built-in conversions:

| Source | Target | Output |
|---|---|---|
| `math::linalg::Matrix<double>` | `std::string` | Each row is wrapped in vertical bars; values are concatenated with `std::to_string`; a newline follows each row |
| `math::linalg::Vector<double>` | `std::string` | Matrix formatting for a single column |
| `math::Point` | `std::string` | `(x, y)` |
| `math::Point` | `math::linalg::Vector<double>` | `{x, y}` |
| `math::Circle` | `std::string` | `Circle at: (x, y) with radius: r` |
| `math::filters::mcl::Particle` | `std::string` | `Particle: <state> with weight: w` |

These string forms are for debugging and logs, not stable serialization. `std::to_string` uses fixed formatting and does not insert separators between matrix values. Only the `double` matrix and vector specializations exist.

You can add a conversion for your own type by specializing `math::conversion` in namespace `math`. Given a type such as `struct MyPoint : math::MathObj<MyPoint> { double x; double y; };`:

```cpp
namespace math {
template <>
struct conversion<MyPoint, std::string> {
    static std::string convert(const MyPoint& point) {
        return "(" + std::to_string(point.x) + ", " +
               std::to_string(point.y) + ")";
    }
};
}
```

The source type must derive from `MathObj<Source>` for `.as<T>()` to be available.
