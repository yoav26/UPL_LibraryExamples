## End-to-End Integration Example

This example shows combined usage of Utility + Motion + CNC modules. For immediate I/O (outside CNC FIFO), see `README_uplIO.md`.

```txt
// 1) Prepare axis/motor
uplMotorOn(A_AXIS)
uplMotorOn(B_AXIS)

// 2) Configure base motion profile
uplMotionProfileConfig(A_AXIS, 100000, 100000, 20000, 0)
uplMotionProfileConfig(B_AXIS, 100000, 100000, 20000, 0)

// 3) Clear CNC queue and configure vector/corner behavior
ACNCAClear
uplCNCSetVectorParams(CNCAPushParam:100, CNCAPushParam:1000000, CNCAPushParam:1000000, CNCAPushParam:0)
uplCNCSetCornerParams(0, ll:1000, 5, 0)

// 4) Queue segments
uplMotionCNCLinear2(A_AXIS, ll:20000, B_AXIS, ll:10000, ll:20000, ll:0)
uplCNCAutomaticCornerMotion(A_AXIS, B_AXIS)
uplMotionCNCLinear2(A_AXIS, ll:0, B_AXIS, ll:0, ll:20000, ll:0)

// 5) Run CNC mode
AMotionMode=11
BMotionMode=11
ABegin
```

### Notes

- Function argument order is user-facing; CNC pushed parameter order is controller-facing.
- Refer to `README_uplCNC.md` for exact segment parameter push order.
