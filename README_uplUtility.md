## uplUtility

Utility helper module for per-axis motor enable / disable commands, scoped per axis through `AChooseAxis[AProgThread]` (the PAxis method).

Source: `uplUtility.pup2` (v5.0.0).

### Naming convention

Utility helpers use the `upl` + action form (utility scope, no controller-keyword segment):

| Pattern | Example |
|---------|---------|
| `upl` + `MotorOn` | `uplMotorOn` |
| `upl` + `MotorOff` | `uplMotorOff` |

### Functions

| Function | Returns | Behavior (after axis select) |
|----------|---------|------------------------------|
| `uplMotorOn(lAxis_)` | — | `PMotorOn = MOTORON_ON` (enable motor) |
| `uplMotorOff(lAxis_)` | — | `PMotorOn = MOTORON_OFF` (disable motor) |

`lAxis_` is the axis index (`A_AXIS` … `H_AXIS`; `0` = A axis, `1` = B axis, and so on).

### Integration Example

```txt
// Enable motor on A axis before running motion
uplMotorOn(A_AXIS)
```

### Constants

- `MOTORON_ON` / `MOTORON_OFF` live in `uphGeneralConstants.puh2` (passed internally by `uplMotorOn` / `uplMotorOff`).
- `uplUtility.puh2` reserves space for utility-local constants; none are currently defined.

### Notes

- APIs are axis-scoped through `AChooseAxis[AProgThread]`.
- Motor on/off values use `MOTORON_ON` / `MOTORON_OFF` from `uphGeneralConstants.puh2`.
