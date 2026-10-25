---
layout: default
title: Tuning
parent: Guides
nav_order: 6
---

# Tuning Guide

| Goal | Setting | Suggested direction |
|---|---|---|
| Reduce persistent PID error | `ki` and `integral_limit` | Increase `ki` gradually; cap accumulated integral if needed |
| Reduce PID overshoot | `kd` | Increase cautiously and filter noisy derivative data |
| Limit mechanism commands | `output_min`, `output_max` | Set safe output bounds for the actuator |
| Trust Kalman measurements more | Lower `R`, or raise `Q` when the model is less certain | Change one assumption at a time |
| Trust the Kalman model more | Raise `R`, or lower `Q` when the model is more certain | Validate against recorded sensor data |
| Preserve more GMM hypotheses | `max_components` | Increase while monitoring compute time |
| Simplify a GMM | `min_weight`, `merge_threshold` | Raise `min_weight` to prune more; raise `merge_threshold` to merge more readily |
| Increase particle-filter coverage | `particle_count` | Increase while monitoring CPU time |
| Resample particles sooner | `resample_ess_ratio` | Increase the threshold |
| Apply a particle weight floor | `min_weight` | Use a small value and verify the model's behavior |

Start with conservative settings, change one parameter at a time, and compare logs from repeatable tests. These directions are general tuning guidance; the correct values depend on the robot, sensor models, units, and update rate.

## Related pages

- [PID control guide]({{ site.baseurl }}/guides/control.html)
- [State estimation guide]({{ site.baseurl }}/guides/state-estimation.html)
- [PID controller API]({{ site.baseurl }}/reference/control.html)
