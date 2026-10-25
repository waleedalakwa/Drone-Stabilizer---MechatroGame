# Mission design

Drone Stabilizer has five sequential stages. The table below summarizes the level configurations programmed into the game.

| # | Mission | Countdown | Gates | Base wind | Gust amplitude | Required gate speed below |
|---|---|---:|---:|---:|---:|---:|
| 01 | Flight Calibration | 100 s | 3 | 0.15 | 0.10 | 2.00 m/s |
| 02 | Crosswind Corridor | 105 s | 3 | 1.15 | 0.45 | 1.75 m/s |
| 03 | Turbulence Alley | 115 s | 4 | 1.45 | 1.35 | 1.75 m/s |
| 04 | Precision Dock | 120 s | 4 | 1.20 | 0.75 | 1.25 m/s |
| 05 | Storm Protocol | 140 s | 5 | 2.10 | 2.25 | 1.55 m/s |

The wind and gust values are coefficients in the environmental-disturbance formula, not measured real-world weather values. Gate radii, positions, and moving-hazard parameters are defined in the `levels` array inside [`index.html`](../index.html).

## Learning progression

**01 — Flight Calibration** introduces checkpoint acquisition and stable position tracking without hazards.

**02 — Crosswind Corridor** adds side wind and fixed hazards. Persistent disturbance highlights the contribution of integral action.

**03 — Turbulence Alley** combines stronger gusts with moving hazards. Derivative damping helps reduce high-speed overshoot near checkpoints.

**04 — Precision Dock** narrows the gates and lowers the allowable entry speed. Accurate, well-damped motion matters more than aggressive tuning.

**05 — Storm Protocol** brings the strongest disturbances and multiple moving hazards together in the final challenge.

## Mission rules and rewards

- Checkpoints must be captured **in order**.
- Capture requires the drone to remain inside a gate at a sufficiently low speed for approximately **0.52 seconds**.
- Each checkpoint adds **150 points**.
- Completing a mission awards a bonus of `round(remaining_seconds × 5 + remaining_integrity × 100)`.
- The game calculates a one-to-three-star mission rating from remaining integrity and, for a three-star result, sufficient remaining time.
- The drone begins each mission with three integrity units. Hazard collisions consume integrity and restore the drone to its last checkpoint.
- Running out of time or integrity ends the attempt; a retry restarts the mission and resets its mission-local score.
- The best score is stored in the browser's local storage where available.

These values document the current game's built-in logic, not a claim of exhaustive performance testing across all missions.
