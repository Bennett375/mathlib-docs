---
layout: default
title: Geometry
parent: References
nav_order: 11
---

# Points, Circles, and Angles

Geometry types keep field calculations explicit. MathLib uses radians for angles. For a task-oriented introduction, see the [field geometry guide]({{ site.baseurl }}/guides/geometry.html).

## `Point` API

| Member | Signature | Behavior |
|---|---|---|
| Constructor | `Point(double x_value = 0.0, double y_value = 0.0)` | Creates a Cartesian point |
| Fields | `double x`, `double y` | Public coordinates |
| Addition | `Point operator+(const Point& other) const` | Component-wise point addition |
| Scalar addition | `Point operator+(double value) const` | Adds `value` to both `x` and `y` |
| Subtraction | `Point operator-(const Point& other) const` | Component-wise point subtraction |
| Scalar subtraction | `Point operator-(double value) const` | Subtracts `value` from both `x` and `y` |
| Equality | `bool operator==(const Point& other) const` | Exact floating-point comparison |
| Inequality | `bool operator!=(const Point& other) const` | Negation of equality |
| Distance | `double distance_to(const Point& other) const` | Euclidean distance via `std::hypot` |
| Rotation | `Point rotated(double radians, const Point& origin = Point()) const` | Counterclockwise rotation around the origin or supplied point |
| Angle | `double angle_to(const Point& other) const` | `atan2` direction in `[-pi, pi]` radians |
| Slope | `double slope_to(const Point& other) const` | `delta_y / delta_x`; a vertical line produces infinite magnitude and identical points produce NaN |

Conversions: `Point::as<std::string>()` produces `(x, y)`, and `Point::as<math::linalg::Vector<double>>()` produces a two-element vector `{x, y}`.

## `Circle`

| Member | Type | Meaning |
|---|---|---|
| `center` | `Point` | Circle center |
| `radius` | `double` | Circle radius |

Constructor: `Circle(const Point& center_, const double& radius_)`. The constructor stores the values as given; it does not reject a negative radius.

`circle_intersections(const Circle& first, const Circle& second)` returns a `std::vector<Point>`:

| Geometric case | Result |
|---|---|
| Centers farther apart than the sum of radii | Empty vector |
| One circle entirely inside the other, with no touching | Empty vector |
| Concentric circles, including identical circles | Empty vector |
| External or internal tangency | One point |
| Two distinct intersection points | Two points |

The function assumes nonnegative radii and valid finite geometry. It uses `std::hypot` and clamps a small negative height to zero, but it is not a general geometry validation layer.

## `angle_error`

`double angle_error(double target, double current)` returns the shortest signed angle from `current` to `target` in `[-pi, pi)`. It wraps with `TAU`, so it is suitable for heading control and for angle residuals in an EKF.

```cpp
double error = math::angle_error(target_heading, current_heading);
```

At an exact half-turn, the result is `-pi`. Treat `+pi` and `-pi` as equivalent when comparing angles.
