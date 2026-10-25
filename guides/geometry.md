---
layout: default
title: Field Geometry
parent: Guides
nav_order: 3
---

# Field Geometry

Geometry types keep field calculations explicit. MathLib uses **radians** for angles.

## Point

`math::Point` stores Cartesian coordinates `x` and `y`. It supports point addition/subtraction, equality comparisons, distance, rotation, angle-to, and slope-to calculations.

```cpp
math::Point robot(24.0, 36.0);
math::Point goal(48.0, 60.0);

double distance = robot.distance_to(goal);
double direction = robot.angle_to(goal);
math::Point rotated_goal = goal.rotated(math::PI / 2.0);
auto coordinates = robot.as<math::linalg::Vector<double>>();
auto text = robot.as<std::string>();
```

`rotated(angle)` rotates around the origin by default. Pass a second `Point` to rotate around a different origin. `angle_to()` returns the direction from the point to the other point; `slope_to()` returns the slope between them.

## Circle and intersections

`math::Circle` stores a center point and radius. `math::circle_intersections(first, second)` returns a vector containing the intersection point or points: typically zero, one, or two.

```cpp
math::Circle c1(math::Point(0.0, 0.0), 10.0);
math::Circle c2(math::Point(12.0, 0.0), 10.0);

std::vector<math::Point> candidates =
    math::circle_intersections(c1, c2);
```

This can generate candidate locations when two known centers and measured distances define two circles. Your localization code must determine which candidate is consistent with the robot's other information.

## Angle helpers

`math::angle_error(target, current)` returns the shortest signed angular difference from the current angle to the target, with wrapping handled by the helper.

```cpp
double error = math::angle_error(
    math::PI / 2.0,  // target: 90 degrees
    3.0 * math::PI / 2.0); // current: 270 degrees
```

Use angle error for heading control so a target near the wrap boundary does not cause an unnecessarily long turn.

## Next step

[Geometry API reference]({{ site.baseurl }}/reference/geometry.html)
