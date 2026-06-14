## uplUtility

Utility helper module for per-axis motor enable/disable commands.

### Integration Example

```txt
// Enable motor on A axis before running motion
uplMotorOn(A_AXIS)
```

### Notes

- APIs are axis-scoped through `AChooseAxis[AProgThread]`.
- Motor on/off values use `MOTORON_ON` / `MOTORON_OFF` from `uphGeneralConstants.puh2` (passed internally by `uplMotorOn` / `uplMotorOff`).
