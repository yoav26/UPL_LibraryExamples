# IDE+ / User Program Guide

A practical reference for the **User Program** environment (**IDE+**) used to write on-drive
programs for Agito controllers. This document focuses on the **language and IDE**, not the
`upl*` library (a short library-rules section is at the end).

> **Scope & confidence:** Everything here is grounded in the sources in this project
> (`*.pup2`, `*.puh2`, `*.puj2`, the generated `*_DefineVars.h`, and `uphGeneralConstants.puh2`)
> plus maintainer clarifications. A few items still pending official confirmation are tagged
> **(to verify)**.

---

## 1. What User Program / IDE+ is

User Program is code that runs **on the drive/controller itself** (not on the PC). You author
it in the **IDE+** editor, compile it, and download it to the drive, where it executes against
the controller's real-time motion/IO engine.

It is an **interpreted scripting language** (run by an on-drive interpreter). The **syntax is
C-based**, with two notable differences:

- **No statement terminator** — there is no trailing `;` on statements.
- **Pythonic indentation** — indentation delimits block bodies. Blocks are opened by
  `function:` / `if` / `switch` / `while` / `for` / `main` and closed by `end` / `endoffunc` /
  `endofmain`.

A program interacts with the controller through **keywords** (built-in symbols such as
`PMotorOn`, `AChooseAxis`, `ACNCAPushType`) rather than a conventional OS API. Two access
styles exist — the **PAxis method** and **direct axis addressing** — described in §8.

---

## 2. File types

| Extension | Role | Tracked in git? |
|-----------|------|-----------------|
| `.puj2` | **Project** file — lists source files and IDE/compiler settings | yes |
| `.pup2` | **Program/implementation** (functions, `main`, logic) | yes |
| `.puh2` | **Header** (`#define`, `#definevar`, shared declarations) | yes |
| `.cup2` / `.cupb2` | **Compiled** program output | no (generated) |
| `*_DefineVars.h` | Generated **symbol → slot/number map** (see §10) | no (generated) |

Generated artifacts (`*.h`, `*.cup2`, `*.cupb2`) are excluded via `.gitignore`.

### 2.1 The project file (`.puj2`)

XML describing the project. Key parts seen in `UPL_LibraryExamples.puj2`:

- `<Files>` — the ordered list of sources compiled into the project.
- `<OpenFiles>` / `<ActiveFile>` — IDE editor state.
- Compiler/editor flags:

| Flag | Meaning **(to verify exact effect)** |
|------|--------------------------------------|
| `AutocompleteEnabled` | Editor autocomplete on/off |
| `ExtendedAutocompleteInfo` | Richer autocomplete details |
| `WarnOnAllocatedGenDataUse` | Warn when using already-allocated `GenData` slots |
| `AllowRedefinition` | Permit redefining symbols |
| `AllowLowLevel` | Permit low-level keywords/operations |
| `AllowLegacyFuncs` | Permit deprecated/legacy functions |
| `NoImpCast` | Disallow implicit casts |
| `AllowImpCast` | Allow implicit casts |
| `AllowImpCastLossy` | Allow implicit casts that may lose precision |

---

## 3. Program structure

A program file typically follows this shape:

```txt
// header banner / comments

#include uphGeneralConstants.puh2      // shared constants first
#include <module headers...>

#information   free-text note shown at build/info time

main([10,30],[5,70],[800,1000],[50,100],[50,100],[50,100])

    // top-level statements run by the main task
    ...
    AProgHaltThis        // optionally halt this program/task

endofmain


// function definitions live outside main
function: ...
endoffunc
```

### 3.1 `main(...)` and allocation ranges

`main` opens the main task body and **declares dynamic-allocation ranges** the compiler uses
to assign resources. The 6 bracketed ranges are (per the in-source comment):

```
main([Tasks],[Functions],[#definevar],[#definevar:f],[#definevar:l],[#definevar:d])
```

Example from this project:

```26:26:UPL_LibraryExamples.pup2
main([10,30],[5,70],[800,1000],[50,100],[50,100],[50,100])
```

| Position | Range here | Allocates |
|----------|-----------|-----------|
| 1 | `[10,30]` | Tasks |
| 2 | `[5,70]` | Functions (each function gets a number — see §10) |
| 3 | `[800,1000]` | `#definevar` (untyped long) → `GenData[]` |
| 4 | `[50,100]` | `#definevar:f` (float) → `GenDataF[]` |
| 5 | `[50,100]` | `#definevar:l` (long) → `GenData[]` family |
| 6 | `[50,100]` | `#definevar:d` (double) → `GenDataD[]` |

Each `[start,end]` is the index window the compiler uses for that category; the body ends with
`endofmain`.

**Why the typed ranges can repeat `[50,100]` without conflict:** each value type is stored in
its **own backing array**, so the *same index in different categories refers to different
storage* and never collides:

| Value type | Backing array |
|------------|---------------|
| `l:` long (32-bit) | `GenData[]` (legacy) |
| `f:` float (32-bit) | `GenDataF[]` |
| `ll:` long-long (64-bit) | `GenDataLL[]` |
| `d:` double (64-bit) | `GenDataD[]` |

So the float/long/double windows all using `[50,100]` is intentional and safe — they resolve to
`GenDataF[]`, `GenData[]`, and `GenDataD[]` respectively. **(to verify)** whether range bounds
are inclusive.

**The number of ranges varies by program.** Only the categories a program actually uses need a
range. Older/simple programs may pass just three — `main([Tasks],[Functions],[#definevar])` — and
add the typed `#definevar:f` / `:l` / `:d` windows only when those variable kinds are declared.
The six-range form above is the fully-typed superset.

### 3.2 Tasks / threads

Programs are **multitasking**. The current thread/task is referenced via `AProgThread`, which is
used to index per-thread state — most importantly the selected axis:
`AChooseAxis[AProgThread]`. The `[Tasks]` allocation range reserves task resources. **(to
verify)** the full task-creation/scheduling API (not exercised in this project).

---

## 4. Preprocessor & declarations

| Directive | Purpose | Example |
|-----------|---------|---------|
| `#include` | Insert another source/header | `#include uplIO.puh2` |
| `#define` | Constant / macro (compile-time substitution) | `#define A_AXIS 0` |
| `#definevar` | Declare a runtime variable (allocated to a slot) | `#definevar testPassCount` |
| `#definevar:f` / `:l` / `:d` | Typed runtime variable (float / long / double) | _(see §3.1)_ |
| `#definevardynamic` | Always-dynamically-allocated global variable; an optional typed tag may precede the name | `#definevardynamic Jerk MyVar_1` |
| `#information` | Emit an informational note at build/info time | `#information See README.md` |

Comments (C-style):

- Line comment: `// ...`
- Block comment: `/* ... */`

### 4.1 Include order (convention used here)

1. `uphGeneralConstants.puh2` (shared constants)
2. Module-local headers
3. Cross-module dependencies only when required

---

## 5. Data types & type prefixes

UPL uses **type prefixes** in two places: variable/return declarations and **typed function
arguments**.

### 5.1 Value types

| Prefix | Type | Backing array | Notes |
|--------|------|---------------|-------|
| `l:` | 32-bit **signed** integer (`long`) | `GenData[]` (legacy) | Signed: `0x80000000` reads as `-2147483648`; `0xFFFFFFFF` as `-1` |
| `ll:` | 64-bit integer (`long long`) | `GenDataLL[]` | Positions/speeds/large values (`ll:1000000`) |
| `f:` | 32-bit floating point (single) | `GenDataF[]` | e.g. analog values: `f:AInValue_` |
| `d:` | 64-bit floating point (double) | `GenDataD[]` | `#definevar:d` allocation category |

Declaration examples:

```txt
function: l:DInPortValue_ = uplIODInPortRead(l:lAxis_)   // l: return + l: arg
function: f:AInValue_ = uplIOAInPortRead(l:lAxis_, l:Index_)
```

### 5.2 Typed argument qualifiers (FW-portable units)

Some function arguments carry a **semantic/unit type tag** instead of a plain value type. These
appear in signatures and at call sites:

| Tag | Used for |
|-----|----------|
| `Speed:` | Speed values `[user-units / sec]` |
| `Accel:` / `Decel:` | Acceleration / deceleration `[user-units / sec^2]` |
| `Jerk:` | Jerk / smoothing `[millisecond]` |
| `AbsTrgt:` | Absolute target position |
| `RelTrgt:` | Relative target distance |
| `CNCAPushParam:` | A value pushed as a CNC FIFO parameter |

Example signature and call:

```40:40:uplMotion.pup2
function: uplMotionProfileConfig(l:lAxis_, Accel:Accel_, Decel:Decel_, Speed:Speed_, Jerk:Jerk_)
```

```txt
uplCNCSetVectorParams(CNCAPushParam:100, CNCAPushParam:1000000, CNCAPushParam:1000000, CNCAPushParam:0)
```

**Why these tags exist:** they make a function **portable across firmware (FW) types** — 32-bit
and 64-bit. The tag tells the compiler how to interpret/scale the value for the target FW, so
the *same* function source compiles correctly for all users regardless of the controller's
numeric width. Use the typed tag (rather than a raw `l:` / `ll:`) wherever a value represents
one of these physical quantities.

### 5.3 Inline type prefixes / tags at call sites and on literals

The same prefixes can be applied **directly to a literal or argument** to force its width or
scaling at the point of use:

- Value-type cast: `ll:10000` treats the literal as 64-bit, e.g.
  `uplMotionPTPAbsolute(B_AXIS, ll:10000)`.
- Typed-tag cast: apply the unit / FW-portable tag to a literal in an assignment or call —
  `PSpeed = Speed: 10000`, `PAccel = Accel: 10000000`, `PJerk = Jerk: 0`.

Typed tags may also appear in `local:` declarations, not just parameters:

```txt
function: uplMotionProfileConfig(..., Speed:Speed_) local: Accel: Accel_2, Speed: Speed_2
```

---

## 6. Functions

```txt
function: <name>(<typed args>) local: <typed locals>
    <body>
endoffunc
```

- **Return value:** prefix the function with a typed result and assign to it:

```txt
function: l:Result_ = name(l:arg_)
    Result_ = ...
endoffunc
```

- **Void function:** omit the result prefix.
- **Locals:** declare after `local:` on the signature line:

```90:90:UPL_LibraryExamples.pup2
function: testDInReadBitMask(l:lAxis_, l:IoMask_) local: l:PortValue_, l:Expected_, l:Actual_
```

- **Calls:** `name(args)`; nested calls are allowed (`Actual_ = uplIODInPortReadBit(lAxis_, IoMask_)`).
- Functions are defined **outside** `main`, and each is assigned a **number** within the
  `[Functions]` allocation range (see §10).
- Parameter naming convention: trailing underscore (`lAxis_`, `IoMask_`).
- **Line continuation:** end a line with `...` to continue a long statement or signature on the
  next line:

```txt
function: uplMotionProfileConfig(l:lAxis_, Accel:Accel_, Decel:Decel_, Speed:Speed_, ...
                                 Jerk:Jerk_) local: Accel: Accel_2, Speed: Speed_2
```

---

## 7. Control flow & operators

### 7.1 Conditionals

```txt
if (<condition>)
    ...
else
    ...
end
```

- Closed by `end`. **Do not** chain `else if` with multiple `end` blocks — nest instead.

### 7.2 Multi-way dispatch

```txt
switch (<expr>)
    case <CONST>
        ...
        break
    case <CONST>
        ...
        break
    default
        break
end
```

Real example:

```80:89:uplIO.pup2
	switch (Action_)
		case IO_WRITE_SET
			PDInLog = PDInLog | IoMask_
			break
		case IO_WRITE_CLEAR
			PDInLog = PDInLog & (IO_MASK_ALL ^ IoMask_)
			break
		default
			break
	end
```

### 7.3 Loops

IDE+ provides `while` and `for` loops. Both are closed by `end`:

```txt
while (<condition>)
    ...
end

for (<init> ; <condition> ; <update>)
    ...
end
```

> Note: no loop is used anywhere in *this* project — repetition here is done via repetitive
> motion modes (e.g. `MOTIONMODE_PTP_REP`) and CNC segments. The loop keywords above are part of
> the language and available for general use.

### 7.4 Operators

The full C-style operator set is supported. The categories below list the common operators; the
examples that happen to appear in this project are not the limit of what's available.

| Category | Operators |
|----------|-----------|
| Assignment | `=` |
| Comparison | `==`, `!=`, `<`, `>`, `<=`, `>=` (and `if (x)` truthiness) |
| Arithmetic | `+`, `-`, `*`, `/` (and modulo) |
| Bitwise | `&`, `\|`, `^` |
| Shift | `<<`, `>>` |
| Logical | logical AND / OR / NOT |

> Don't confuse language operators with CNC *trigger constants* such as `CNC_TRIG_GT` /
> `CNC_TRIG_LT`, which select greater/less-than behavior **inside the CNC engine** (they are
> constants, not operators).

---

## 8. The `A*` vs `P*` keyword model (core concept)

Controller state is reached through **keywords**. There are two complementary styles.

### 8.1 PAxis method (axis selected per thread)

1. Select the target axis for the current thread:
   ```txt
   AChooseAxis[AProgThread] = lAxis_
   ```
2. Use `P*` keywords, which act on the **currently selected axis**:

```45:50:uplIO.pup2
function: l:DInPortValue_ = uplIODInPortRead(l:lAxis_)
	
	AChooseAxis[AProgThread] = lAxis_
	DInPortValue_ = PDInPort
	
```

This is the dominant pattern across the whole `upl*` library. Examples of `P*` keywords seen:
`PDInPort`, `PDInLog`, `PDInMode[]`, `PDOutPort`, `PDOutLog`, `PDOutMode[]`, `PDOutSelect[]`,
`PDOutType`, `PAInPort[]`, `PAInMode[]`, `PAOutPort[]`, `PAOutMode[]`, `PMotorOn`, `PAccel`,
`PDecel`, `PSpeed`, `PJerk`, `PMotionMode`, `PRelTrgt`, `PAbsTrgt`, `PRptWait`, `PWaitStatus[]`,
`PBegin`, `PStop`.

### 8.2 Direct axis addressing (axis letter prefix)

A per-axis keyword can be addressed for a **specific axis** by prefixing it with the axis letter.
The library defines constants for eight axes (`A_AXIS`…`H_AXIS`), but the keyword namespace
extends to as many axes as the system supports — test programs use letters past `H` (e.g.
`JDOutPort`, and `CDOutPort` / `DDOutPort` / `EDOutPort` for axes C / D / E). Seen in the CNC
example:

```56:58:UPL_LibraryExamples.pup2
// AMotionMode=11
// BMotionMode=11
// ABegin
```

Here `AMotionMode` / `BMotionMode` set motion mode on axis A / B, and `ABegin` starts axis A.

### 8.3 Axis-related vs global keywords

**Not every keyword is axis-related** — whether the leading letter means "axis A" depends on the
keyword:

- **Axis-related keywords** have a per-axis instance and are addressed per axis — via the
  axis-letter prefix (`AMotionMode`, `BMotionMode`, `ABegin`) or via the `P*` form on the
  `AChooseAxis`-selected axis. `UserParam` / `UserParamLL` are **axis-related** (per-axis).
- **Global (non-axis) keywords** exist once for the whole program/engine; the leading letter is
  just part of the name, not an axis selector. `GenData` / `GenDataLL` / `GenDataF` / `GenDataD`
  are **global**. Also global: `AProgThread` (current thread), `AWaitTime` (delay),
  `AProgHaltThis` (halt this program), and the CNC engine keywords `ACNCAClear`,
  `ACNCAPushType`, `ACNCAPushParam`, `ACNCADoStep`.

> Rule of thumb: per-axis quantities (motion mode, targets, IO ports, `UserParam`) are
> axis-scoped; program/engine-wide data (`GenData`, thread/program control, the CNC queue) is
> global.

---

## 9. Built-in commands & runtime

| Keyword / command | Purpose | Example |
|-------------------|---------|---------|
| `printf(fmt, args...)` | Console/log output with formatting | `printf("!G!val=%l", x)` |
| `AWaitTime, <ms>` | Delay | `AWaitTime, 500` |
| `AProgHaltThis` | Halt the current program/task | — |
| `PBegin` / `<axis>Begin` | Start motion on selected / specific axis | `PBegin`, `ABegin` |
| `PStop` | Stop motion on selected axis | — |
| `PWaitStatus[<type>], <value>` | Block until a status condition is met | `PWaitStatus[MOTION_STATUS], MOTION_END` |
| `<axis>SetPosition, <value>` | Set (redefine) the current position of an axis | `BSetPosition, 0` |

### 9.1 `printf` formatting

- **Format specifiers:** `%l` for `l:` integers (confirmed). `%f` / `%d` for float/double are
  **(to verify)**.
- **Markup / color codes:** a leading code colors the line — `!G!` = green, `!R!` = red
  (confirmed in tests). Other codes (e.g. yellow/blue) are **(to verify)**.

```98:98:UPL_LibraryExamples.pup2
		printf("!G!uplIODInPortReadBit PASS axis=%l mask=%l expected=%l actual=%l", lAxis_, IoMask_, Expected_, Actual_)
```

---

## 10. Variables & memory model

- `#definevar NAME` declares a runtime variable; the compiler binds it to a **slot** inside the
  matching allocation window. From the generated map:

  ```7:8:UPL_LibraryExamples_DefineVars.h
  #define testPassCount "AGenData[800]"	//From file: upl_libraryexamples.pup2
  #define testFailCount "AGenData[801]"	//From file: upl_libraryexamples.pup2
  ```

  i.e. an untyped `#definevar` becomes an entry in the global **`GenData[]`** array (`AGenData[]`
  is the keyword form).

- Typed `#definevar` variables go to their type's backing array (see §3.1): `:f` → `GenDataF[]`,
  `:l` → `GenData[]`, `:d` → `GenDataD[]`; 64-bit integers use `GenDataLL[]`.

- `#define NAME value` is a plain compile-time substitution:

  ```13:13:UPL_LibraryExamples_DefineVars.h
  #define EXAMPLE_TARGET_POSITION "10000"	//From file: upl_libraryexamples.pup2
  ```

- **Functions** are also assigned numbers in the `[Functions]` window (generated map shows e.g.
  `#define uplIODInModeConfig <n>`).

- Controller data arrays: `GenData[]` (32-bit) / `GenDataLL[]` (64-bit) and the floating
  variants `GenDataF[]` (32-bit) / `GenDataD[]` (64-bit) are **global**; `UserParam[]` (32-bit) /
  `UserParamLL[]` (64-bit) are **per-axis**.

> Tip: `WarnOnAllocatedGenDataUse` helps catch accidental reuse of `GenData[]` slots that the
> compiler already allocated to a `#definevar`.

---

## 11. CNC FIFO model (brief)

Motion/IO can be **queued** into the CNC engine's FIFO and executed in sequence. Mechanics
observed:

- `ACNCAClear` — clear the CNC FIFO/queue.
- `ACNCAPushType, <typeword>` — push a **segment header**. The 32-bit type word packs the
  segment type in the high byte and involved-axis nibbles in the low bytes:

  ```txt
  (CNC_SEGMENT_TYPE_LINEAR << 24) | ((lAxis1_ & 0xF) << 20) | ... | <unused-mask>
  ```

- `ACNCAPushParam, <value>` — push the segment's parameters, **in a controller-defined order**
  (which may differ from the user-facing function argument order — see `README_uplCNC.md`).
- Run by switching to a CNC motion mode (`AMotionMode = 11`, i.e. `MOTIONMODE_CNCA`) and issuing
  `ABegin`.

Segment types, parameter orders, and trigger constants live in `uplCNC.puh2` /
`uphGeneralConstants.puh2` and are detailed in `README_uplCNC.md`.

---

## 12. Build & version control

- **Build:** open the `.puj2` in IDE+, compile (produces `.cup2` / `.cupb2` and refreshes
  `*_DefineVars.h`), download to the drive, and run.
- After renaming/adding functions or `#definevar`s, **rebuild** so `*_DefineVars.h` (the
  symbol→number/slot map) is regenerated.
- **Git:** commit only sources (`.puj2`, `.pup2`, `.puh2`, docs). Generated files are ignored:

  ```1:7:.gitignore
  # Agito / UPL generated and build artifacts
  *.h
  *.cup2
  *.cupb2
  ```

---

## 13. Concise UPL library rules

These govern the `upl*` helper modules built on top of IDE+ (full spec in
`LIBRARY_CONVENTIONS.md`).

- **Naming:** `upl<Module><Keyword><Action>` (no separators) — e.g. `uplIODInPortReadBit`,
  `uplIODOutPortWriteBit`, `uplIODInModeConfig`.
- **Files:** `*.puh2` headers, `*.pup2` implementations, `upl*` prefix for library files,
  `uph*` for shared constants.
- **Axis-first:** user-facing APIs take `lAxis_` first; functions select the axis via
  `AChooseAxis[AProgThread] = lAxis_` (PAxis method).
- **Config vs Read vs Write:** config-only keywords (`DInMode[]`, `DOutMode[]`, `DOutSelect[]`,
  `DOutType`) expose only `*Config` / `*Set` helpers; ports expose Read/Write (full + per-bit).
- **Per-bit I/O:** pass `IO_NUM_n` masks with `IO_WRITE_SET` / `IO_WRITE_CLEAR`.
- **Constants ownership:** global constants in `uphGeneralConstants.puh2`; module-local
  constants in the module's `.puh2`.
- **Docs & examples:** every public function carries a UPL-style comment block (title +
  `// -- arg_ : description` lines); examples must match real signatures.
- **Versioning:** semantic `MAJOR.MINOR.PATCH` in each module header; classify changes as
  behavioral vs non-behavioral in the changelog.
