## uplMotion

Motion helper module for single-axis profile setup and PTP / JOG motion, scoped per axis through `AChooseAxis[AProgThread]` (the PAxis method).

Source: `uplMotion.pup2` (v5.0.0). Section order in source: **Profile config → PTP (relative / absolute) → Repetitive PTP (relative / absolute) → JOG → Wait / Stop**.

### Naming convention

Motion helpers use the `uplMotion` + action form (legacy style; `uplWait2EndOfMotion` predates the prefix and is kept for backward compatibility):

| Pattern | Example |
|---------|---------|
| `uplMotion` + `ProfileConfig` | `uplMotionProfileConfig` |
| `uplMotion` + `PTPRelative` | `uplMotionPTPRelative` |
| `uplMotion` + `PTPAbsolute` | `uplMotionPTPAbsolute` |
| `uplMotion` + `PTPRelativeRep` | `uplMotionPTPRelativeRep` |
| `uplMotion` + `PTPAbsoluteRep` | `uplMotionPTPAbsoluteRep` |
| `uplMotion` + `JOG` | `uplMotionJOG` |
| `uplMotion` + `Stop` | `uplMotionStop` |
| (legacy) | `uplWait2EndOfMotion` |

### Functions

Listed in **`uplMotion.pup2` source order**.

**Profile configuration**

| Function | Returns | Behavior (after axis select) |
|----------|---------|------------------------------|
| `uplMotionProfileConfig(lAxis_, Accel_, Decel_, Speed_, Jerk_)` | — | `PAccel = Accel_`, `PDecel = Decel_`, `PSpeed = Speed_`, `PJerk = Jerk_` |

**Point-to-point motion**

| Function | Returns | Behavior (after axis select) |
|----------|---------|------------------------------|
| `uplMotionPTPRelative(lAxis_, TargetPos_)` | — | `PMotionMode = MOTIONMODE_PTP`, `PRelTrgt = TargetPos_`, `PBegin` |
| `uplMotionPTPAbsolute(lAxis_, TargetPos_)` | — | `PMotionMode = MOTIONMODE_PTP`, `PAbsTrgt = TargetPos_`, `PBegin` |

**Repetitive point-to-point motion**

| Function | Returns | Behavior (after axis select) |
|----------|---------|------------------------------|
| `uplMotionPTPRelativeRep(lAxis_, TargetPos_, lRepWait_)` | — | `PMotionMode = MOTIONMODE_PTP_REP`, `PRptWait = lRepWait_`, `PRelTrgt = TargetPos_`, `PBegin` |
| `uplMotionPTPAbsoluteRep(lAxis_, TargetPos_, lRepWait_)` | — | `PMotionMode = MOTIONMODE_PTP_REP`, `PRptWait = lRepWait_`, `PAbsTrgt = TargetPos_`, `PBegin` |

**JOG (continuous) motion**

| Function | Returns | Behavior (after axis select) |
|----------|---------|------------------------------|
| `uplMotionJOG(lAxis_, Speed_)` | — | `PMotionMode = MOTIONMODE_JOG`, `PSpeed = Speed_`, `PBegin` |

**Wait / Stop**

| Function | Returns | Behavior (after axis select) |
|----------|---------|------------------------------|
| `uplWait2EndOfMotion(lAxis_)` | — | `PWaitStatus[MOTION_STATUS], MOTION_END` (blocks until motion completes) |
| `uplMotionStop(lAxis_)` | — | `PStop` |

### Argument types & units

Signatures use typed argument tags so the same source is FW-portable across 32/64-bit controllers:

| Argument | Tag in signature | Units |
|----------|------------------|-------|
| `Accel_` | `Accel:` | `[user-units / sec^2]` |
| `Decel_` | `Decel:` | `[user-units / sec^2]` |
| `Speed_` | `Speed:` | `[user-units / sec]` (signed for JOG) |
| `Jerk_` | `Jerk:` | `[millisecond]` (smoothing time) |
| `TargetPos_` (absolute) | `AbsTrgt:` | `[user-units]` |
| `TargetPos_` (relative) | `RelTrgt:` | `[user-units]` |
| `lRepWait_` | `l:` | `[millisecond]` (dwell at each target) |
| `lAxis_` | `l:` | axis index (`A_AXIS` … `H_AXIS`) |

### Constants (`uplMotion.puh2`)

- `MOTION_STATUS` (7) and `MOTION_END` (0) — wait-status alias / value used by `uplWait2EndOfMotion`.
- Motion-mode constants (`MOTIONMODE_PTP`, `MOTIONMODE_PTP_REP`, `MOTIONMODE_JOG`, …) live in `uphGeneralConstants.puh2`.

### Integration Example

```txt
// Configure and run one absolute move on A axis
uplMotionProfileConfig(A_AXIS, 100000, 100000, 20000, 0)
uplMotionPTPAbsolute(A_AXIS, 50000)
uplWait2EndOfMotion(A_AXIS)
```

### Notes

- APIs are axis-scoped through `AChooseAxis[AProgThread]`.
- Units are controller / user-units based, as documented in function comments.
- PTP / JOG / repetitive helpers issue `PBegin` internally; `uplMotionProfileConfig` only sets profile keywords and does **not** start motion.
