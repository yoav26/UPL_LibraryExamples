## uplMotion

Motion helper module for single-axis profile setup and PTP/JOG commands.

### Integration Example

```txt
// Configure and run one absolute move on A axis
uplMotionProfileConfig(A_AXIS, 100000, 100000, 20000, 0)
uplMotionPTPAbsolute(A_AXIS, 50000)
uplWait2EndOfMotion(A_AXIS)
```

### Notes

- APIs are axis-scoped through `AChooseAxis[AProgThread]`.
- Units are controller/user-units based, as documented in function comments.
