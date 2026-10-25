---
layout: default
title: Random Numbers
parent: References
nav_order: 10
---

# Random Numbers

`math::random` provides the `RandomSource` interface and a seeded `Rng` implementation for simulation and filter testing. For a task-oriented introduction, see the [arithmetic and random guide]({{ site.baseurl }}/guides/arithmetic-and-random.html).

## API

| Type or member | Signature | Behavior |
|---|---|---|
| `RandomSource` | Abstract base class | Virtual destructor; declares the two sampling methods below |
| `RandomSource::uniform01` | `virtual double uniform01() = 0` | Returns a uniform value in `[0, 1)` |
| `RandomSource::normal` | `virtual double normal(double mean, double standard_deviation) = 0` | Returns a normal sample with the requested mean and standard deviation; the standard deviation should be positive |
| `Rng` | `Rng(int seed = 0)` | `std::mt19937`-backed implementation |
| `Rng::uniform01` | `double uniform01() override` | Uniform sample in `[0, 1)` |
| `Rng::normal` | `double normal(double mean, double standard_deviation) override` | Normal sample; the second argument is the standard deviation, so the variance is its square |

## Seeding

Seed behavior:

- `seed == 0` uses `std::random_device` to request a fresh seed.
- A nonzero seed gives a repeatable sequence on the same platform and standard-library implementation.
- A custom `RandomSource` is useful for deterministic tests, recorded noise playback, or a different random engine.

```cpp
class FixedRandom : public math::random::RandomSource {
public:
    double uniform01() override {
        return 0.5;
    }

    double normal(double, double) override {
        return 0.0;
    }
};
```

`Rng` and `RandomSource` implementations are not required to be thread-safe. Use a separate source per task or add synchronization. Neither the interface nor `Rng` validates the standard deviation; pass a positive value.
