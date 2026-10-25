# PID controller and flight model

Drone Stabilizer is designed as a **two-dimensional position-control sandbox**. Its visual drone is driven by a simulated planar acceleration command. The model intentionally omits rotor-level dynamics and assumes the inner flight-control system and hover-thrust compensation are already handled.

## State and control law

For each axis (`x` and `y`):

- `p`: simulated position in meters
- `r`: target (setpoint) position in meters
- `v`: corresponding simulated velocity in m/s
- `e = r - p`: position tracking error
- `I`: accumulated position error, limited to `[-6, 6]`

The acceleration command before saturation is:

```text
u_x = Kp * (target_x - x) + Ki * I_x - Kd * vx
u_y = Kp * (target_y - y) + Ki * I_y - Kd * vy
```

The derivative contribution is calculated from the measured velocity (`-Kd * v`) instead of directly differentiating a changing setpoint. This reduces *derivative kick* when a player moves the target abruptly.

The game's default preset is **Balanced: `Kp=2.20`, `Ki=0.35`, `Kd=2.40`**. The tuning interface labels the implied dimensional units as `Kp: s⁻²`, `Ki: s⁻³`, and `Kd: s⁻¹`.

## Saturation and integral handling

After computing the provisional command, the integrator is updated conditionally to reduce windup under saturation. Each axis's integral state is clamped between **−6 and +6**. If `Ki=0`, both integral states reset to zero.

The **vector magnitude** of the commanded acceleration is then capped at **8.5 m/s²**. These are game model parameters, not measured limits from a physical drone.

## Planar dynamics

The acceleration on each axis is the sum of the bounded controller command, a varying environmental disturbance, and linear damping:

```text
ax = ux + wind_x - 0.38 * vx
ay = uy + wind_y - 0.38 * vy
```

Position and velocity are stepped using **semi-implicit Euler integration** at a fixed simulation interval of **1/120 second**:

```text
v_next = v_now + a * dt
p_next = p_now + v_next * dt
```

The total velocity is limited to **8.2 m/s**. The simulated world is approximately **24 m × 13.5 m**, mapped to a 1200 × 675 Canvas using 50 pixels per world unit. The aircraft is also constrained to stay within the gameplay boundaries.

## Disturbances and hazards

Mission-dependent base wind and gust amplitudes feed deterministic time- and position-varying sine/cosine disturbances. Hazard positions may also oscillate; contact triggers a simulated collision, temporary invulnerability, and a checkpoint reset. These mechanics provide game challenges rather than a validated aerodynamic environment.

## Completion and observability

A gate captures the drone only if the distance to its center is less than its mission-specific radius and drone speed is below its mission-specific threshold. Those conditions must persist for approximately **0.52 seconds**. Otherwise, the gate-hold timer decays.

The telemetry panel displays:

- **Position error:** magnitude of `target - position` in meters.
- **Speed:** magnitude of planar velocity in m/s.
- **Wind force:** magnitude of the modeled disturbance, labeled `m/s²` in the interface.
- **Control effort:** percentage of available commanded-acceleration magnitude.
- **Tracking-error chart:** the most recent **up to 110 samples**, sampled roughly every `0.11 s` (approximately 12 seconds total).

The game's render loop uses `requestAnimationFrame`; physics steps are taken in fixed increments when actively playing.

## Important limitations

This model **does not** implement 3D flight mechanics, rotation matrices, rigid-body attitude integration, propulsion/rotor models, sensor noise, physical motor saturation, or experimentally identified hardware parameters. The game coordinates and disturbances serve a pedagogical control-system simulation; observed performance should not be treated as evidence of a tuned flight controller for real hardware.
