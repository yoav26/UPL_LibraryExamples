## uplIO

Digital and analog I/O helpers, scoped per axis through `AChooseAxis[AProgThread]`.

Source: `uplIO.pup2` (v5.0.0). Section order in source: **Digital inputs → Digital outputs → Analog input → Analog output**; within each section: **Config → Read → Write**.

### Naming convention

Functions follow **`uplIO` + controller keyword + action**:

| Pattern | Example |
|---------|---------|
| `uplIO` + `DInPort` + `Read` | `uplIODInPortRead` |
| `uplIO` + `DInPort` + `ReadBit` | `uplIODInPortReadBit` |
| `uplIO` + `DInLog` + `Write` | `uplIODInLogWrite` (full 32-bit port) |
| `uplIO` + `DInLog` + `WriteBit` | `uplIODInLogWriteBit` (per-bit SET/CLEAR) |
| `uplIO` + `DOutPort` + `Read` | `uplIODOutPortRead` |
| `uplIO` + `DOutPort` + `ReadBit` | `uplIODOutPortReadBit` |
| `uplIO` + `DOutPort` + `Write` | `uplIODOutPortWrite` (full 32-bit port) |
| `uplIO` + `DOutPort` + `WriteBits` | `uplIODOutPortWriteBits` (per-bit SET/CLEAR) |
| `uplIO` + `DOutLog` + `Write` | `uplIODOutLogWrite` (full 32-bit port) |
| `uplIO` + `DOutLog` + `WriteBit` | `uplIODOutLogWriteBit` (per-bit SET/CLEAR) |
| `uplIO` + `DInMode` + `Config` | `uplIODInModeConfig` |
| `uplIO` + `DOutMode` + `Config` | `uplIODOutModeConfig` |
| `uplIO` + `DOutSelect` + `Config` | `uplIODOutSelectConfig` |
| `uplIO` + `DOutType` + `Config` | `uplIODOutTypeConfig` |
| `uplIO` + `AInMode` + `Config` | `uplIOAInModeConfig` |
| `uplIO` + `AInPort` + `Read` | `uplIOAInPortRead` |
| `uplIO` + `AOutMode` + `Config` | `uplIOAOutModeConfig` |
| `uplIO` + `AOutPort` + `Read` | `uplIOAOutPortRead` |
| `uplIO` + `AOutPort` + `Write` | `uplIOAOutPortWrite` |

**Config-only keywords** (`DInMode[]`, `DOutMode[]`, `DOutSelect[]`, `DOutType`): uplIO provides **Config** helpers only — no Read wrappers. Use controller keywords directly if readback is needed.

**Config** functions set `DInMode[]`, `DOutMode[]`, `DOutSelect[]`, or `DOutType`.

**Full-port write** functions assign the entire 32-bit value: `uplIODInLogWrite`, `uplIODOutPortWrite`, `uplIODOutLogWrite`.

**Per-bit write** functions take `IO_NUM_n` and `IO_WRITE_SET` / `IO_WRITE_CLEAR`: `uplIODInLogWriteBit`, `uplIODOutPortWriteBits`, `uplIODOutLogWriteBit`.

**Toggle** functions XOR a mask: `uplIODInLogToggleBits`, `uplIODOutPortToggleBits`, `uplIODOutLogToggleBits`.

### Controller digital I/O model

| Keyword | Type | Index / bit rule |
|---------|------|------------------|
| `DInPort` | 32-bit | bit *n* → input #(*n*+1); input #1 = bit 0 |
| `DInLog` | 32-bit | same bit mapping as `DInPort`; **program write** (bitwise) via `uplIO` |
| `DInMode[]` | 32 × | array index → input # (**1-based**); **config only** |
| `DOutPort` | 32-bit | bit *n* → output #(*n*+1); output #1 = bit 0 |
| `DOutLog` | 32-bit | same bit mapping as `DOutPort`; **program write** (bitwise) via `uplIO` |
| `DOutMode[]` | 32 × | array index → output # (**1-based**); **config only** |
| `DOutSelect[]` | 16 × | array index → output # (**1-based**); **config only** |
| `DOutType` | 32-bit | per-bit: `DOUTTYPE_SINK` (0) / `DOUTTYPE_SOURCE` (1); **config only** |

UPL access: `PDInPort`, `PDInLog`, `PDInMode[]`, `PDOutPort`, `PDOutLog`, `PDOutMode[]`, `PDOutSelect[]`, `PDOutType` after axis selection.

### Per-bit I/O constants (`uplIO.puh2`)

Pass **`IO_NUM_n`** (bit mask for I/O #n) as `IoMask_` to ReadBit, WriteBits, and ToggleBits helpers. Pass **`IO_WRITE_SET`** or **`IO_WRITE_CLEAR`** to WriteBits helpers.

| Constant | Value | Use |
|----------|-------|-----|
| `IO_NUM_1` … `IO_NUM_32` | single-bit mask | pass directly as `IoMask_` |
| `IO_WRITE_SET` | 0 | OR bit on (`\|= IO_NUM_n`) |
| `IO_WRITE_CLEAR` | 1 | clear bit (`&= IO_MASK_ALL ^ IO_NUM_n`) |
| `IO_MASK_ALL` | `0xFFFFFFFF` | used internally for CLEAR |

Example: `uplIODInLogWriteBit(A_AXIS, IO_NUM_1, IO_WRITE_SET)`

### Integration example

```
uplIODOutModeConfig(A_AXIS, 1, DOUTMODE_USER_OUTPUT)
uplIODOutPortWrite(A_AXIS, 0)
uplIODOutPortWriteBits(A_AXIS, IO_NUM_1, IO_WRITE_SET)
AWaitTime, 500
uplIODOutPortWriteBits(A_AXIS, IO_NUM_1, IO_WRITE_CLEAR)

l:din = uplIODInPortRead(A_AXIS)
if (uplIODInPortReadBit(A_AXIS, IO_NUM_1))
	uplIODInLogWriteBit(A_AXIS, IO_NUM_1, IO_WRITE_SET)
```

### Digital functions

Listed in **`uplIO.pup2` source order**.

**Digital inputs — Config**

| Function | Returns | Maps to |
|----------|---------|---------|
| `uplIODInModeConfig(lAxis_, InputIndex_, Mode_)` | — | `PDInMode[InputIndex_]` |

**Digital inputs — Read**

| Function | Returns | Maps to |
|----------|---------|---------|
| `uplIODInPortRead(lAxis_)` | `l` | `PDInPort` |
| `uplIODInPortReadBit(lAxis_, IoMask_)` | `l` | `PDInPort & IoMask_` (pass `IO_NUM_n`) |

**Digital inputs — Write (DInLog)**

| Function | Returns | Maps to |
|----------|---------|---------|
| `uplIODInLogWrite(lAxis_, DInLogValue_)` | — | `PDInLog` (full 32-bit value) |
| `uplIODInLogWriteBit(lAxis_, IoMask_, Action_)` | — | SET: `\|= IoMask_`; CLEAR: `&= IO_MASK_ALL ^ IoMask_` |
| `uplIODInLogToggleBits(lAxis_, IoMask_)` | — | `PDInLog ^= IoMask_` |

**Digital outputs — Config**

| Function | Returns | Maps to |
|----------|---------|---------|
| `uplIODOutModeConfig(lAxis_, OutputIndex_, Mode_)` | — | `PDOutMode[OutputIndex_]` |
| `uplIODOutSelectConfig(lAxis_, SelectIndex_, SelectValue_)` | — | `PDOutSelect[SelectIndex_]` |
| `uplIODOutTypeConfig(lAxis_, DOutTypeValue_)` | — | `PDOutType` |

**Digital outputs — Read**

| Function | Returns | Maps to |
|----------|---------|---------|
| `uplIODOutPortRead(lAxis_)` | `l` | `PDOutPort` |
| `uplIODOutPortReadBit(lAxis_, IoMask_)` | `l` | `PDOutPort & IoMask_` (pass `IO_NUM_n`) |

**Digital outputs — Write (DOutPort)**

| Function | Returns | Maps to |
|----------|---------|---------|
| `uplIODOutPortWrite(lAxis_, DOutPortValue_)` | — | `PDOutPort` (full 32-bit value) |
| `uplIODOutPortWriteBits(lAxis_, IoMask_, Action_)` | — | SET: `\|= IoMask_`; CLEAR: `&= IO_MASK_ALL ^ IoMask_` |
| `uplIODOutPortToggleBits(lAxis_, IoMask_)` | — | `PDOutPort ^= IoMask_` |

**Digital outputs — Write (DOutLog)**

| Function | Returns | Maps to |
|----------|---------|---------|
| `uplIODOutLogWrite(lAxis_, DOutLogValue_)` | — | `PDOutLog` (full 32-bit value) |
| `uplIODOutLogWriteBit(lAxis_, IoMask_, Action_)` | — | SET: `\|= IoMask_`; CLEAR: `&= IO_MASK_ALL ^ IoMask_` |
| `uplIODOutLogToggleBits(lAxis_, IoMask_)` | — | `PDOutLog ^= IoMask_` |

Mode constants: `DINMODE_*`, `DOUTMODE_*` in `uphGeneralConstants.puh2`.

### Analog functions

**Analog input** (Config → Read)

| Function | Returns | Maps to |
|----------|---------|---------|
| `uplIOAInModeConfig(lAxis_, InputIndex_, Mode_)` | — | `PAInMode[InputIndex_]` |
| `uplIOAInPortRead(lAxis_, Index_)` | `f` | `PAInPort[Index_]`; **`Index_` > 0** |

**Analog output** (Config → Read → Write)

| Function | Returns | Maps to |
|----------|---------|---------|
| `uplIOAOutModeConfig(lAxis_, OutputIndex_, Mode_)` | — | `PAOutMode[OutputIndex_]` |
| `uplIOAOutPortRead(lAxis_, Index_)` | `f` | `PAOutPort[Index_]`; **`Index_` > 0** |
| `uplIOAOutPortWrite(lAxis_, Index_, AOutValue_)` | — | `PAOutPort[Index_]`; **`Index_` > 0** |

Analog mode constants: `ANALOG_*` in `uphGeneralConstants.puh2` (shared with `AInMode` / `AOutMode`).

### Constants (`uplIO.puh2` / `uphGeneralConstants.puh2`)

**`uplIO.puh2`**

- `IO_NUM_1` … `IO_NUM_32` — single-bit masks by I/O number (`IO_NUM_32` = `IO_NUM_31 << 1`)
- `IO_WRITE_SET`, `IO_WRITE_CLEAR`, `IO_MASK_ALL`

**`uphGeneralConstants.puh2`**

- `DINMODE_*`, `DOUTMODE_*`, `DOUTTYPE_SINK` / `DOUTTYPE_SOURCE`
- `IO_DIN_COUNT` (32), `IO_DOUT_COUNT` (32), `IO_DOUT_SELECT_COUNT` (16)
- `DINPORT_DINn_BIT`, `DOUTPORT_DOUTn_BIT` / `_SET` / `_CLEAR` (legacy)
