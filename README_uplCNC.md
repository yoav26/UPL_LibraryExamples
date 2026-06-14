## uplCNC CNC Segment Specifications

This document specifies the CNC segment interface implemented in `uplCNC.pup2`.  
All CNC segments are emitted to the CNC FIFO as:

- **CNCAPushType**: 32‑bit word  
  - Bits 31–24: **segment type** (see **Segment Types Overview** below)  
  - Bits 23–0: **involved axes mask** – six 4‑bit slots (1st axis = bits 20–23, …, 6th axis = bits 0–3)  
  - Uninvolved axis slots must be 0xF, via the `*_UNUSED_MASK` macros.
- **CNCAPushParam**: one or more parameter pushes, in the exact order specified for each segment type.


## Segment Types Overview

- **Type 1 – Linear** (`CNC_SEGMENT_TYPE_LINEAR = 1`)  
  Linear motion segments with 1–6 involved axes. Parameters: target positions for involved axes, followed by `Speed` and `EndSpeed`.

- **Type 2 – Arc** (`CNC_SEGMENT_TYPE_ARC = 2`)  
  Circular arc in a plane defined by two axes, with center, direction and speeds.

- **Type 5 – Set Vector Parameters** (`CNC_SEGMENT_TYPE_SET_VECTOR_PARAMS = 5`)  
  Global vector speed/accel configuration (no involved axes).

- **Type 6 - Set Corner Parameters** (`CNC_SEGMENT_TYPE_CORNER_PARAMS = 6`)  
  Configures automatic cornering behavior for ARC corners (radius/error model, angle threshold, and axis-acceleration limiting).

- **Type 8 - DOUT Port Write** (`CNC_SEGMENT_TYPE_DOUTPORT_WRITE = 8`)  
  Writes a digital output port value; `lAxis_` is pushed as data (not encoded in involved-axis bits).

- **Type 9 - Array Write** (`CNC_SEGMENT_TYPE_ARRAY_WRITE = 9`)  
  Writes a value into a controller array. No axes are involved in `CNCAPushType`; `lAxis_` is pushed as a normal parameter.

- **Type 12 - Wait Using User Array** (`CNC_SEGMENT_TYPE_WAIT_ARRAY = 12`)  
  Waits, with all member axes in-position, until a trigger condition based on `GenData[]` or `UserParam[]` is satisfied.

- **Type 14 - Automatic Corner Motion** (`CNC_SEGMENT_TYPE_AUTOMATIC_CORNER = 14`)  
  Inserts an automatic corner between linear segments; controller validates the previous segment and required dummy parameters.

- **Type 17 - Multiple Write to User Array** (`CNC_SEGMENT_TYPE_ARRAY_WRITE_MULTIPLE = 17`)  
  Performs four consecutive assignments into `GenData[]` or axis-specific `UserParam[]` when the CNC motion reaches this segment.

- **Type 18 - Multiple Write to User Array and Wait Using User Array** (`CNC_SEGMENT_TYPE_ARRAY_WRITE_MULTIPLE_WAIT = 18`)  
  Performs the same four assignments as Type 17, then waits using trigger definitions on the same selected array at `TriggerIndexValue`.

- **Type 21 - Wait Using Inputs** (`CNC_SEGMENT_TYPE_WAIT_INPUT = 21`)  
  Waits (with all member axes in-position) for a trigger condition on either `DInPort` or `AInPort`.

- **Type 22 – Spatial Events Setup** (`CNC_SEGMENT_TYPE_SPATIAL_EVENTS_SETUP = 22`)  
  Configures which axis generates spatial events and their pulse characteristics.

- **Type 23 – Spatial Events** (`CNC_SEGMENT_TYPE_SPATIAL_EVENTS = 23`)  
  Defines up to 15 event positions on the previously configured axis.


## Linear Motion Segments (Type 1)

All linear helpers push `CNCAPushType` directly with `CNC_SEGMENT_TYPE_LINEAR` in bits 31–24 and involved axes packed into the 4-bit slots, then push:

1. Target positions for each involved axis, in the order of the function arguments
2. `Speed_` (cruise speed)
3. `EndSpeed_` (end‑of‑segment speed)

Uninvolved axis slots are filled using the appropriate `*_UNUSED_MASK` macro.

- **Function**: `uplMotionCNCLinear1`  
  - **Axes**: exactly 1  
  - **Signature**:  
    - `l:lAxis1_` – axis index (`A_AXIS`, `B_AXIS`, …)  
    - `AbsTrgt:TrgtPos1_` – absolute target position (axis 1)  
    - `Speed:Speed_` – cruise speed  
    - `Speed:EndSpeed_` – end speed  
  - **CNCAPushType**: `CNC_SEGMENT_TYPE_LINEAR` + first‑axis slot, all others uninvolved (`ONEINV_UNUSED_MASK`)  
  - **Params (ACNCAPushParam)**: `TrgtPos1_`, `Speed_`, `EndSpeed_`

- **Function**: `uplMotionCNCLinear2`  
  - **Axes**: exactly 2  
  - **Signature**:  
    - `l:lAxis1_`, `AbsTrgt:TrgtPos1_`  
    - `l:lAxis2_`, `AbsTrgt:TrgtPos2_`  
    - `Speed:Speed_`, `Speed:EndSpeed_`  
  - **CNCAPushType**: type 1 + axis 1 in slot1, axis 2 in slot2, remaining via `TWOINV_UNUSED_MASK`  
  - **Params**: `TrgtPos1_`, `TrgtPos2_`, `Speed_`, `EndSpeed_`

- **Function**: `uplMotionCNCLinear3`  
  - **Axes**: exactly 3  
  - **Signature**: `lAxis1_`, `TrgtPos1_`, `lAxis2_`, `TrgtPos2_`, `lAxis3_`, `TrgtPos3_`, `Speed_`, `EndSpeed_`  
  - **CNCAPushType**: type 1 + axes in slots 1–3, `THREEINV_UNUSED_MASK` for remaining  
  - **Params**: `TrgtPos1_`, `TrgtPos2_`, `TrgtPos3_`, `Speed_`, `EndSpeed_`

- **Function**: `uplMotionCNCLinear4`  
  - **Axes**: exactly 4  
  - **Signature**:  
    - `l:lAxis1_`, `AbsTrgt:TrgtPos1_`  
    - `l:lAxis2_`, `AbsTrgt:TrgtPos2_`  
    - `l:lAxis3_`, `AbsTrgt:TrgtPos3_`  
    - `l:lAxis4_`, `AbsTrgt:TrgtPos4_`  
    - `Speed:Speed_`, `Speed:EndSpeed_`  
  - **CNCAPushType**: type 1 + axes in slots 1–4, `FOURINV_UNUSED_MASK` for remaining  
  - **Params**: `TrgtPos1_`..`TrgtPos4_`, `Speed_`, `EndSpeed_`

- **Function**: `uplMotionCNCLinear5`  
  - **Axes**: exactly 5  
  - **Signature**:  
    - `l:lAxis1_`, `AbsTrgt:TrgtPos1_`  
    - `l:lAxis2_`, `AbsTrgt:TrgtPos2_`  
    - `l:lAxis3_`, `AbsTrgt:TrgtPos3_`  
    - `l:lAxis4_`, `AbsTrgt:TrgtPos4_`  
    - `l:lAxis5_`, `AbsTrgt:TrgtPos5_`  
    - `Speed:Speed_`, `Speed:EndSpeed_`  
  - **CNCAPushType**: type 1 + axes in slots 1–5, `FIVEINV_UNUSED_MASK` for remaining  
  - **Params**: `TrgtPos1_`..`TrgtPos5_`, `Speed_`, `EndSpeed_`

- **Function**: `uplMotionCNCLinear6`  
  - **Axes**: exactly 6  
  - **Signature**:  
    - `l:lAxis1_`, `AbsTrgt:TrgtPos1_`  
    - `l:lAxis2_`, `AbsTrgt:TrgtPos2_`  
    - `l:lAxis3_`, `AbsTrgt:TrgtPos3_`  
    - `l:lAxis4_`, `AbsTrgt:TrgtPos4_`  
    - `l:lAxis5_`, `AbsTrgt:TrgtPos5_`  
    - `l:lAxis6_`, `AbsTrgt:TrgtPos6_`  
    - `Speed:Speed_`, `Speed:EndSpeed_`  
  - **CNCAPushType**: type 1 + axes in all six slots (no unused mask macro)  
  - **Params**: `TrgtPos1_`..`TrgtPos6_`, `Speed_`, `EndSpeed_`

- **Function (planned)**: `uplMotionCNCLinear` (unified 1–5 axis helper)  
  - **Status**: placeholder; implementation described in comments only.  
  - **Intended behavior**: accepts `TrgtPos1_`..`TrgtPos5_`, treats a sentinel value (e.g. 0xF) as “no axis”, packs involved axes into slots in order, and pushes only the active targets followed by `Speed_` and `EndSpeed_`.


## Arc Motion Segment (Type 2)

- **Segment type**: `CNC_SEGMENT_TYPE_ARC = 2`
- **Function**: `uplCNCArcMotion`  
  - **Axes**: 2 involved axes (plane of the arc)  
  - **Signature**:  
    - `l:lAxis1_`, `l:lAxis2_` – plane axes  
    - `AbsTrgt:TrgtPos1_`, `AbsTrgt:TrgtPos2_` – arc end position in the plane  
    - `AbsTrgt:CenterPos1`, `AbsTrgt:CenterPos2` – center location in the same plane  
    - `l:Dir` – direction (e.g. CW/CCW encoded)  
    - `Speed:cruise_speed` – cruise speed along the path  
    - `Speed:end_speed` – end‑of‑segment speed  
  - **CNCAPushType**: type 2 + axis1 in slot1, axis2 in slot2, remaining via `TWOINV_UNUSED_MASK`  
  - **Params** (in order):  
    - `TrgtPos1_`, `TrgtPos2_`, `CenterPos1`, `CenterPos2`, `Dir`, `cruise_speed`, `end_speed`, `ll:0` (no additional cycles), `CNCAPushParam:0` (Dummy1), `ll:0` (Dummy2)


## Vector Parameter Segment (Type 5)

- **Segment type**: `CNC_SEGMENT_TYPE_SET_VECTOR_PARAMS = 5`
- **Function**: `uplCNCSetVectorParams`  
  - **Axes**: 0 involved axes (`NONINV_UNUSED_MASK`)  
  - **Signature**:  
    - `CNCAPushParam:SpeedPercentsInternal_` – internal speed percentage  
    - `CNCAPushParam:VectorAcceleration_` – vector acceleration \([user‑units / sec²]\)  
    - `CNCAPushParam:VectorDeceleration_` – vector deceleration \([user‑units / sec²]\)  
    - `CNCAPushParam:VectorJerk_` – vector jerk \([user‑units / sec³]\)  
  - **Params**: `SpeedPercentsInternal_`, `VectorAcceleration_`, `VectorDeceleration_`, `VectorJerk_`

## Corner Parameter Segment (Type 6)

- **Segment type**: `CNC_SEGMENT_TYPE_CORNER_PARAMS = 6`
- **Function**: `uplCNCSetCornerParams`  
  - **Axes**: 0 involved axes (`NONINV_UNUSED_MASK`)  
  - **Signature**:  
    - `l:CornerRadiusMethod_`  
    - `ll:CornerRadiusOrError_`  
    - `l:MinimalAngleForCorner_`  
    - `l:AxisAccelerationLimitType_`  
  - **Hidden/internal pushed params**:
    - `CornerType` is always pushed as `1` (ARC)
    - `CornerAxisAcceleration` is always pushed as `0`
  - **Params** (in order):  
    - `ll:1` (fixed `CornerType = ARC`)  
    - `CornerRadiusMethod_`  
    - `CornerRadiusOrError_`  
    - `ll:0` (fixed `CornerAxisAcceleration = 0`)  
    - `MinimalAngleForCorner_`  
    - `AxisAccelerationLimitType_`

### Corner parameter semantics and constraints

- **CornerType**  
  - `1` = ARC (round) corner.  
  - `2` = Continuous Acceleration corner (**not supported**, ignore).  
  - Current API always sends `1`.

- **CornerRadiusMethod**  
  - Relevant only for ARC (`CornerType = 1`, which is fixed by current API).  
  - `0` = parameter is direct corner radius.  
  - `1` = parameter is maximal corner error (controller derives radius from corner angle).  
  - Current API does not expose Continuous Acceleration mode.

- **CornerRadiusOrError**  
  - Used for ARC corners (current API mode).  
  - If `CornerRadiusMethod = 0`: radius in counts.  
  - If `CornerRadiusMethod = 1`: maximal corner error in counts.  
  - Must be positive (non-zero).

- **CornerAxisAcceleration**  
  - Nominal axis peak acceleration [counts/sec2] for Continuous Acceleration corners.  
  - Must be `0` for ARC.  
  - Must be positive (non-zero) for Continuous Acceleration.
  - Current API always sends `0` and does not expose this parameter.

- **MinimalAngleForCorner**  
  - Minimal angle [degrees] between two linear segments required to create a corner.  
  - If actual angle is less than or equal to this value, no corner is executed (sharp reference velocity change may occur).  
  - Valid range: `[0, 180]`.

- **AxisAccelerationLimitType**  
  - `0` = do not reduce corner speed due to per-axis acceleration limits.  
  - `1` = reduce automatic corner motion speed based on user-provided axis acceleration limits.

### Cornering behavior notes

- Corners are added only if the user pushes an Automatic Corner Motion segment between the two relevant linear motion segments.
- If no Automatic Corner Motion segment is pushed, the controller does not add a corner.

### Internal defaults (until first Set Corner Parameters segment)

- `CornerType = 1`
- `CornerRadiusMethod = 0`
- `CornerRadiusOrError = 1000`
- `CornerAxisAcceleration = 0`
- `MinimalAngleForCorner = 5`
- `AxisAccelerationLimitType = 0`

## DOUT Port Write Segment (Type 8)

- **Segment type**: `CNC_SEGMENT_TYPE_DOUTPORT_WRITE = 8`
- **Function**: `uplCNCDOutPortWrite`  
  - **Axes**: 0 involved axes (`NONINV_UNUSED_MASK`)  
  - **Purpose**: write a digital output port value with an axis reference carried as parameter data.  
  - **Signature**:  
    - `l:lAxis_` – axis reference passed as parameter data  
    - `l:DOutPortValue_` – digital output port value to write  
  - **CNCAPushType**: type 8 + no involved axes (`NONINV_UNUSED_MASK`)  
  - **Params**: `lAxis_`, `DOutPortValue_`

## Array Write Segment (Type 9)

- **Segment type**: `CNC_SEGMENT_TYPE_ARRAY_WRITE = 9`
- **Function**: `uplCNCArrayWrite`  
  - **Axes**: 0 involved axes (`NONINV_UNUSED_MASK`)  
  - **Purpose**: write a value into a controller array entry.  
  - **Special note**: `lAxis_` is not encoded into the involved-axis bits; it is pushed as a normal parameter.  
  - **Signature**:  
    - `l:lAxis_` – axis reference passed as data for the array write; used for `UserParam[]` selection  
    - `l:ArrayType_` – array selection (`0` = `GenData[]`, `1` = `UserParam[]`; supported systems use `GenDataLL[]` and `UserParamLL[]`)  
    - `l:Index_` – target array index value  
    - `l:Value_` – value to write  
  - **CNCAPushType**: type 9 + no involved axes (`NONINV_UNUSED_MASK`)  
  - **Params**: `ArrayType_`, `lAxis_`, `Index_`, `Value_`
  - **Behavior when executed**:  
    - If `ArrayType_ = 0`: `GenData[Index_] = Value_`  
    - If `ArrayType_ = 1`: `<lAxis_>UserParam[Index_] = Value_`

## Wait Using User Array Segment (Type 12)

- **Segment type**: `CNC_SEGMENT_TYPE_WAIT_ARRAY = 12`
- **Function**: `uplCNCWaitUsingUserArray`  
  - **Axes**: 0 involved axes (`NONINV_UNUSED_MASK`)  
  - **Purpose**: when the CNC engine reaches this segment, it waits, in-position of all member axes, until the configured trigger condition is satisfied.  
  - **Signature**:  
    - `l:lAxis_` – axis identification used for `UserParam[]` only (`0` = A axis, `1` = B axis, and so on)  
    - `l:ArrayType_` – array selection (`0` = `GenData[]`, `1` = `UserParam[]`)  
    - `l:Index_` – target array index value  
    - `l:TriggerType_` – trigger type selector  
    - `l:TriggerValue_` – trigger comparison value; ignored for `TriggerType_ = 8` but still must be pushed  
  - **CNCAPushType**: type 12 + no involved axes (`NONINV_UNUSED_MASK`)  
  - **Params**: `ArrayType_`, `lAxis_`, `Index_`, `TriggerType_`, `TriggerValue_`
  - **Ordering note**: function args are `lAxis_`-first for convenience; pushed `Params` order is controller-defined.

### Trigger type values

- `0`: None (CNC engine will not wait for a condition, but see note below)
- `1`: Greater than
- `2`: Equal To
- `3`: Not Equal To
- `4`: Smaller Than
- `5`: Rising Edge
- `6`: Falling Edge
- `7`: Reserved (acts as `TriggerType_ = 0`)
- `8`: Upon change (`TriggerValue_` is ignored, but must still be included in the pushed segment)

### Wait behavior notes

- `ArrayType_ = 0` refers to `GenData[]`, and `ArrayType_ = 1` refers to `UserParam[]`.
- `lAxis_` is used only for the `UserParam[]` case.
- The segment halts CNC segment execution for at least one sample time, even if the trigger condition is already met immediately or `TriggerType_ = 0`.

## Automatic Corner Motion Segment (Type 14)

- **Segment type**: `CNC_SEGMENT_TYPE_AUTOMATIC_CORNER = 14`
- **Function**: `uplCNCAutomaticCornerMotion`  
  - **Axes**: 2 involved axes (`TWOINV_UNUSED_MASK`)  
  - **Purpose**: request the controller to insert an automatic corner motion between linear segments (corner behavior is configured by Type 6 Corner Params).  
  - **Signature**:  
    - `l:lAxis1_`, `l:lAxis2_` – the two involved axes (must match previous motion segment involved axes)  
  - **CNCAPushType**: type 14 + axis1 in slot1, axis2 in slot2, remaining via `TWOINV_UNUSED_MASK`  
  - **Params**: `CNCAPushParam:0` (Dummy1), `ll:0` (Dummy2) … `ll:0` (Dummy10) — **10 total**, all must be `0`

### Controller validation checks

- **Dummy parameter checks**: Exactly 10 parameters must be present, and all must be 0.
- **Previous segment type**: The last motion segment must be a Linear Motion segment.
- **Axes consistency**: The involved axes in this segment must be identical to the involved axes of the previous motion segment.

## Multiple Write to User Array Segment (Type 17)

- **Segment type**: `CNC_SEGMENT_TYPE_ARRAY_WRITE_MULTIPLE = 17`
- **Function**: `uplCNCArrayWriteMultiple`  
  - **Axes**: 0 involved axes (`NONINV_UNUSED_MASK`)  
  - **Purpose**: force multiple assignments of user-defined `GenData[]` or `UserParam[]` elements with user-defined values at the specific time when CNC motion reaches this segment.  
  - **Signature**:  
    - `l:lAxis_` – axis identification used for `UserParam[]` only (`0` = A axis, `1` = B axis, and so on)  
    - `l:ArrayType_` – array selection (`0` = `GenData[]`, `1` = `UserParam[]`)  
    - `l:Index_` – starting array index value  
    - `l:Value1_`, `l:Value2_`, `l:Value3_`, `l:Value4_` – values written to four consecutive elements  
  - **CNCAPushType**: type 17 + no involved axes (`NONINV_UNUSED_MASK`)  
  - **Params**: `ArrayType_`, `lAxis_`, `Index_`, `Value1_`, `Value2_`, `Value3_`, `Value4_`
  - **Ordering note**: function args are `lAxis_`-first for convenience; pushed `Params` order is controller-defined.

### Behavior when executed

- If `ArrayType_ = 0`:
  - `GenData[Index_] = Value1_`
  - `GenData[Index_ + 1] = Value2_`
  - `GenData[Index_ + 2] = Value3_`
  - `GenData[Index_ + 3] = Value4_`
- If `ArrayType_ = 1`:
  - `<lAxis_>UserParam[Index_] = Value1_`
  - `<lAxis_>UserParam[Index_ + 1] = Value2_`
  - `<lAxis_>UserParam[Index_ + 2] = Value3_`
  - `<lAxis_>UserParam[Index_ + 3] = Value4_`

## Multiple Write to User Array and Wait Using User Array Segment (Type 18)

- **Segment type**: `CNC_SEGMENT_TYPE_ARRAY_WRITE_MULTIPLE_WAIT = 18`
- **Function**: `uplCNCArrayWriteMultipleWait`  
  - **Axes**: 0 involved axes (`NONINV_UNUSED_MASK`)  
  - **Purpose**: first performs multiple assignments to `GenData[]` or `UserParam[]` (same as Type 17), then waits in-position of all member axes until trigger condition is satisfied.  
  - **Signature**:  
    - `l:lAxis_` – axis identification used for `UserParam[]` only (`0` = A axis, `1` = B axis, and so on)  
    - `l:ArrayType_` – array selection (`0` = `GenData[]`, `1` = `UserParam[]`)  
    - `l:Index_` – starting index value for write operations  
    - `l:Value1_`, `l:Value2_`, `l:Value3_`, `l:Value4_` – values written to four consecutive elements  
    - `l:TriggerIndex_` – index value used for wait trigger evaluation  
    - `l:TriggerType_` – trigger type selector  
    - `l:TriggerValue_` – trigger comparison value; ignored for `TriggerType_ = 8` but must still be pushed  
  - **CNCAPushType**: type 18 + no involved axes (`NONINV_UNUSED_MASK`)  
  - **Params**: `ArrayType_`, `lAxis_`, `Index_`, `Value1_`, `Value2_`, `Value3_`, `Value4_`, `TriggerIndex_`, `TriggerType_`, `TriggerValue_`
  - **Ordering note**: function args are `lAxis_`-first for convenience; pushed `Params` order is controller-defined.

### Behavior when executed

- First, performs the same 4 consecutive write operations as Type 17:
  - If `ArrayType_ = 0`:
    - `GenData[Index_] = Value1_`
    - `GenData[Index_ + 1] = Value2_`
    - `GenData[Index_ + 2] = Value3_`
    - `GenData[Index_ + 3] = Value4_`
  - If `ArrayType_ = 1`:
    - `<lAxis_>UserParam[Index_] = Value1_`
    - `<lAxis_>UserParam[Index_ + 1] = Value2_`
    - `<lAxis_>UserParam[Index_ + 2] = Value3_`
    - `<lAxis_>UserParam[Index_ + 3] = Value4_`
- Then waits according to trigger definitions, using the same selected array (`GenData[]` or `UserParam[]`) with `TriggerIndex_`.

### Trigger type values

- `0`: None (CNC engine will not wait for a condition, but see note below)
- `1`: Greater than
- `2`: Equal To
- `3`: Not Equal To
- `4`: Smaller Than
- `5`: Rising Edge
- `6`: Falling Edge
- `7`: Reserved (acts as `TriggerType_ = 0`)
- `8`: Upon change (`TriggerValue_` is ignored, but must still be included in the pushed segment)

### Wait behavior note

- This segment halts CNC segment execution for at least one sample time, even if the trigger condition is already met immediately or `TriggerType_ = 0`.

## Wait Using Inputs Segment (Type 21)

- **Segment type**: `CNC_SEGMENT_TYPE_WAIT_INPUT = 21`
- **Function**: `uplCNCWaitInput`  
  - **Axes**: 0 involved axes (`NONINV_UNUSED_MASK`)  
  - **Purpose**: when this segment is reached, the CNC engine waits (in-position of all member axes) until the configured trigger condition is satisfied based on selected input source.  
  - **Signature**:  
    - `l:lAxis_` – axis identification (`0` = A axis, `1` = B axis, and so on)  
    - `l:InputSelection_` – input selection (`0` = `DInPort`, `1` = `AInPort`)  
    - `l:Index_` – index used for `AInPort[Index_]` (used for `AInPort` only; must be greater than 0)  
    - `ll:DInputsMask_` – mask applied to `DInPort` before trigger check (used for `DInPort` only; default value `-1`)  
    - `l:TriggerType_` – trigger type selector  
    - `CNCAPushParam:TriggerValue_` – trigger comparison value; ignored for `TriggerType_ = 8` but still must be pushed  
  - **CNCAPushType**: type 21 + no involved axes (`NONINV_UNUSED_MASK`)  
  - **Params**: `InputSelection_`, `lAxis_`, `Index_`, `DInputsMask_`, `TriggerType_`, `TriggerValue_`
  - **Ordering note**: function args are `lAxis_`-first for convenience; pushed `Params` order is controller-defined.

### Input and trigger semantics

- `InputSelection_ = 0` refers to `DInPort`; `InputSelection_ = 1` refers to `AInPort`.
- For `DInPort`, trigger is evaluated after AND with `DInputsMask_`.
- For `AInPort`, trigger is evaluated on `AInPort[Index_]`.
- `TriggerValue_` supports any value.

### Trigger type values

- `0`: None (CNC engine will not wait for a condition, but see note below)
- `1`: Greater than
- `2`: Equal To
- `3`: Not Equal To
- `4`: Smaller Than
- `5`: Rising Edge
- `6`: Falling Edge
- `7`: Reserved (acts as `TriggerType_ = 0`)
- `8`: Upon change (`TriggerValue_` is ignored, but must still be included in the pushed segment)

### Wait behavior note

- The `Wait Using Inputs` segment halts CNC segment execution for at least one sample time, even if the trigger condition is already met immediately or `TriggerType_ = 0`.

## Spatial Events Setup Segment (Type 22)

- **Segment type**: `CNC_SEGMENT_TYPE_SPATIAL_EVENTS_SETUP = 22`
- **Function**: `uplCNCSpatialEventsSetup`  
  - **Axes**: 0 involved axes (`NONINV_UNUSED_MASK`)  
  - **Purpose**: configure which axis generates spatial events and their pulse characteristics.  
  - **Signature**:  
    - `l:EventAxis_` – axis index generating events (0=A, 1=B, 2=C, …)  
    - `l:IsVirtualAxis_` – 0 = physical axis, 1 = virtual axis  
    - `l:IsPreconfiguredTbl_` – 0 = not preconfigured, 1 = preconfigured table  
    - `l:EventPulseRes_` – event pulse resolution  
    - `l:EventPulseWidth_` – event pulse width  
    - `l:EventSelect_` – event selection / mode  
  - **Params**:  
    - `EventAxis_`, `IsVirtualAxis_`, `IsPreconfiguredTbl_`, `EventPulseRes_`, `EventPulseWidth_`, `EventSelect_`


## Spatial Events Segment (Type 23)

- **Segment type**: `CNC_SEGMENT_TYPE_SPATIAL_EVENTS = 23`
- **Function**: `uplCNCSpatialEvents`  
  - **Axes**: 0 involved axes (`NONINV_UNUSED_MASK`)  
  - **Purpose**: define up to 15 spatial event positions on the axis configured by `uplCNCSpatialEventsSetup`.  
  - **Behavior**:  
    - Segment is **not motion‑blocking**.  
    - All events share the same select/pulse‑width configuration from the setup segment.  
  - **Signature**:  
    - `l:NumEvents_` – number of events (1..15)  
    - `AbsTrgt:EventPos1_` … `AbsTrgt:EventPos15_` – event positions along the configured axis  
  - **Params** (17 total):  
    1. `NumEvents_`  
    2. `EventPos1_`  
    3. `EventPos2_`  
    4. `EventPos3_`  
    5. `EventPos4_`  
    6. `EventPos5_`  
    7. `EventPos6_`  
    8. `EventPos7_`  
    9. `EventPos8_`  
    10. `EventPos9_`  
    11. `EventPos10_`  
    12. `EventPos11_`  
    13. `EventPos12_`  
    14. `EventPos13_`  
    15. `EventPos14_`  
    16. `EventPos15_`  
    17. `ll:0` – dummy parameter, must be 0 (used internally).


## CNC Control Commands (non‑segment helpers)

These helpers manipulate CNC control flags/keywords rather than emitting motion segments:

- **Function**: `uplCNCSetSpeedPercent`  
  - **Signature**: `l:percent_`  
  - Sets `ACNCAPercents` directly (`ACNCAPercents = percent_`).
  - No range clamp/validation is performed by this helper.

- **Function**: `uplCNCStop`  
  - Sets `StopCNCA` to stop CNC motion.

- **Function**: `uplCNCPause`  
  - Sets `CNCAPause = 1` to pause CNC motion.

- **Function**: `uplCNCResume`  
  - Sets `CNCAPause = 0` to resume CNC motion.

- **Function**: `uplCNCStepModeOn`  
  - Sets `CNCAStepMode = 1` to enable step mode.

- **Function**: `uplCNCDoStep`  
  - Sets `CNCADoStep = 1` to perform a single CNC step when in step mode.

- **Function**: `uplCNCStepModeOff`  
  - Sets `CNCAStepMode = 0` to disable step mode.


## Notes and Conventions

- All axis references (`lAxis#_`, `EventAxis_`) are integer indices; use the predefined `*_AXIS` constants where possible.
- All absolute target positions (`AbsTrgt:*`) are expressed in **user units**.
- When adding new segment types, follow the same pattern:
  - Define a `CNC_SEGMENT_TYPE_*` constant in `uplCNC.puh2`.
  - Push `CNCAPushType` directly with that type and the correct axis slots / unused mask.
  - Push parameters strictly in the order defined by the spec using `ACNCAPushParam`.
