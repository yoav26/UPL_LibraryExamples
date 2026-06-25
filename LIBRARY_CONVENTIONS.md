# User Program Library Conventions

This document is the single source of truth for conventions used in this library package:

- `uphGeneralConstants.puh2`
- `uplMotion.puh2`
- `uplMotion.pup2`
- `uplUtility.puh2`
- `uplUtility.pup2`
- `uplCNC.puh2`
- `uplCNC.pup2`
- `uplIO.puh2`
- `uplIO.pup2`
- module readmes (currently `README_uplCNC.md`, `README_uplIO.md`, `README_uplMotion.md`, `README_uplUtility.md`)

---

## 1) Naming and File Conventions

- **Header files** use `*.puh2`; implementation files use `*.pup2`.
- **UPL library files** use `upl*` prefix.
- **General shared constants file** uses `uph*` prefix.
- **Function names** use module prefix, controller keyword, then action:
  - Pattern: `upl<Module><Keyword><Action>` (no separators)
  - I/O examples: `uplIODInPortRead`, `uplIODInPortReadBit`, `uplIODOutPortReadBit`, `uplIODOutPortWrite`, `uplIODInModeConfig`
  - Motion: `uplMotion...` (legacy; migrate to keyword pattern when touched)
  - Utility: `upl...` (utility scope)
  - CNC: `uplCNC...` (segment-oriented; `uplCNCDOutPortWrite` already matches keyword + action)
- **Constants/macros** use `UPPER_SNAKE_CASE`.
- **Function parameters** use trailing underscore (`_`), e.g. `lAxis_`, `ArrayType_`, `TriggerValue_`.
- **Axis arguments** should use `lAxis_` naming consistently across CNC/Motion/Utility APIs unless a function already has an established public signature.

## 2) Function Signature Conventions

- Keep public signatures stable once published.
- Use explicit type prefix in signatures (`l:`, `ll:`, `Speed:`, `AbsTrgt:`, `CNCAPushParam:`) consistent with existing module style.
- Parameter order must be documented exactly as implemented.
- Any intentional mismatch between user-facing arg order and controller-facing pushed order must be explicitly documented.

## 3) Axis-First API Rule (CNC user-facing APIs)

- For user convenience, CNC APIs that include axis identification should place `lAxis_` first in function arguments when practical and backward-compatible.
- Example (user-facing): `uplCNCWaitUsingUserArray(lAxis_, ArrayType_, Index_, TriggerType_, TriggerValue_)`

## 4) Push-Order Semantics (Controller-facing)

- `ACNCAPushType` and `ACNCAPushParam` order is defined by controller protocol and must not be inferred from argument order.
- Every CNC API section must include:
  - **Signature** (function argument order)
  - **Params** (actual pushed parameter order)
  - **Ordering note** when these differ
- Do not reorder pushed params unless protocol changes are explicitly approved.

## 5) Include Order

Use this include order in `*.pup2` files:

1. Shared/general constants (`uphGeneralConstants.puh2`)
2. Module-local headers
3. Cross-module dependencies (only when required)

Example:

```txt
#include uphGeneralConstants.puh2
#include uplMotion.puh2
#include uplUtility.puh2
```

## 6) Comments and Documentation Format

- **UPL control flow** (IDE/compiler grammar):
  - Use `if` / `else` / `end` — do not chain `else if` with multiple `end` blocks.
  - For multi-way integer dispatch (e.g. trigger type selectors), prefer `switch` / `case` / `break` / `default` / `end`.
- Header banner first line must match actual file name exactly.
- Use consistent section labels:
  - `Segment type value`
  - `Number of involved axes`
  - `Func. args`
  - `Params`
  - `Behavior when executed`
- Use consistent wording and spelling:
  - `reference`, `absolute`, `position`, `millisecond`, `constants`, `array`
- Units must be explicit where applicable (`[user-units / sec]`, etc.).
- Use `lAxis_`, `ArrayType_`, `TriggerType_`, etc. consistently in docs/comments.

## 7) Constants Ownership (Canonical Source)

- **Global/common constants** belong in `uphGeneralConstants.puh2`:
  - Axis IDs (`A_AXIS`...`H_AXIS`)
  - Platform-wide IO/event/fault constants (`DINMODE_*`, `DOUTMODE_*`, `DOUTTYPE_*`, port bit masks)
  - Any cross-module enums/macros reused by multiple modules
- **Module-specific constants** belong in module headers:
  - `uplMotion.puh2`: motion-library-local constants (e.g., wait status aliases used by motion helpers)
  - `uplUtility.puh2`: utility-local constants (reserved; motor on/off values are in `uphGeneralConstants.puh2`)
  - `uplIO.puh2`: I/O bit masks (`IO_NUM_n`, `IO_WRITE_SET`, `IO_WRITE_CLEAR`); legacy port masks remain in `uphGeneralConstants.puh2`
  - `uplCNC.puh2`: CNC segment IDs, masks, CNC wait-segment trigger/input/array type constants (`CNC_TRIG_*`)
- Avoid duplicate active definitions across files unless aliasing is intentional and documented.

## 8) Versioning Convention

- Use semantic versioning in module headers: `MAJOR.MINOR.PATCH`.
- **PATCH**: typo/doc/comment-only or non-functional cleanup.
- **MINOR**: backward-compatible feature/documentation expansion.
- **MAJOR**: breaking API/signature/behavior changes.
- For coordinated library release, all touched modules should be version-reviewed together.

## 9) Change Classification for Releases

Every release note/changelog entry should classify each change as:

- `No behavioral change`
- `Behavioral change`

If behavioral, include impact and migration guidance.

## 10) Validation Policy

- Prefer lightweight runtime guards for high-risk APIs where feasible and non-disruptive.
- If no runtime guard is added, docs must clearly state invalid input expectations:
  - `undefined behavior if ...`
  - or equivalent explicit precondition statements.

## 11) Example Quality Rules

- Examples must match actual implemented function signatures.
- Examples must reflect actual pushed parameter order notes where relevant.
- Numeric examples in comments must match called values in code.
- Keep one minimal working example per API section in docs when possible.

---

## Adoption Notes

- This file governs conventions for future changes.
- Existing legacy style may remain temporarily for backward compatibility, but any touched area should be normalized to this spec as part of ongoing maintenance.
