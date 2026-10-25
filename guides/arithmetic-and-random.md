---
layout: default
title: Arithmetic and Random
parent: Guides
nav_order: 5
---

# Arithmetic and Random

## Arithmetic utilities

The `math` namespace includes constants and helpers for common calculations.

| Utility | Purpose | Example |
|---|---|---|
| `PI`, `TAU`, `e` | Common mathematical constants | `double full_turn = math::TAU;` |
| `mean(values)` | Mean of a vector of doubles; returns `0` for an empty vector | `math::mean({1.0, 2.0, 3.0})` |
| `mean(a, b)` | Mean of two values | `math::mean(4.0, 8.0)` |
| `sign(value)` | Returns `-1`, `0`, or `1` | `math::sign(-3.0)` |
| `clamp(value, min, max)` | Restricts a value to an inclusive interval | `math::clamp(120.0, -100.0, 100.0)` |
| `power(base, exponent)` | Raises a base to a real exponent | `math::power(2.0, 3.0)` |
| `exp(value)` | Natural exponential | `math::exp(1.0)` |
| `sqr(value)` / `sqrt(value)` | Square / square root | `math::sqrt(math::sqr(3.0))` |
| `make_groups(values, size, disjoint)` | Splits a sequence into groups of at most `size` | `math::make_groups(values, 3)` |
| `make_pairs(values, disjoint)` | Splits a sequence into pairs | `math::make_pairs(values, false)` |

For `make_groups`, `disjoint=true` advances by the group size. With `false`, the window advances by one element, so groups can overlap by more than one element when the group size is larger than two. A group size of zero returns no groups.

## Random numbers and simulation

`math::random::Rng` implements the `RandomSource` interface, which supplies uniform values in `[0, 1)` and normal samples with a requested mean and standard deviation.

```cpp
math::random::Rng repeatable_rng(1234);
double unit_sample = repeatable_rng.uniform01();
double sensor_noise = repeatable_rng.normal(0.0, 0.5);
```

A non-zero seed gives a repeatable sequence on the same platform. The default seed `0` requests a fresh seed. Filters that accept `RandomSource` can also use a custom source, which is useful for controlled tests or simulation.

## API reference pages

- [Arithmetic and conversion API]({{ site.baseurl }}/reference/arithmetic.html)
- [Random API]({{ site.baseurl }}/reference/random.html)
