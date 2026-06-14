## uplIO

Digital and analog I/O helpers, scoped per axis through `AChooseAxis[AProgThread]`.

### Naming convention

Functions follow **`uplIO` + controller keyword + action**:

| Pattern | Example |
|---------|---------|
| `uplIO` + `DInPort` + `Read` | `uplIODInPortRead` |
| `uplIO` + `DInPort` + `ReadBit` | `uplIODInPortReadBit` |
| `uplIO` + `DOutPort` + `Write` | `uplIODOutPortWrite` (per-bit) |
| `uplIO` + `DInMode` + `Config` | `uplIODInModeConfig` |
| `uplIO` + `DOutSelect` + `Read` | `uplIODOutSelectRead` |

Digital **Config** functions set `DInMode[]`, `DOutMode[]`, `DOutSelect[]`, or `DOutType`.  
**WritePort** functions assign the full 32-bit port value.  
Per-bit **Write** functions take `IO_NUM_n` and `IO_WRITE_SET` / `IO_WRITE_CLEAR`.

### Controller digital I/O model

| Keyword | Type | Index / bit rule |
|---------|------|------------------|
| `DInPort` | 32-bit | bit *n* → input #(*n*+1); input #1 = bit 0 |
| `DInLog` | 32-bit | same bit mapping as `DInPort`; **program write** (bitwise) via `uplIO` |
| `DInMode[]` | 32 × | array index → input # (**1-based**) |
| `DOutPort` | 32-bit | bit *n* → output #(*n*+1); output #1 = bit 0 |
| `DOutLog` | 32-bit | same bit mapping as `DOutPort` |
| `DOutMode[]` | 32 × | array index → output # (**1-based**) |
| `DOutSelect[]` | 16 × | array index → output # (**1-based**) |
| `DOutType` | 32-bit | per-bit: `DOUTTYPE_SINK` (0) / `DOUTTYPE_SOURCE` (1) |

UPL access: `PDInPort`, `PDInLog`, `PDInMode[]`, `PDOutPort`, `PDOutLog`, `PDOutMode[]`, `PDOutSelect[]`, `PDOutType` after axis selection.

### Per-bit I/O constants (`uplIO.puh2`)

Pass **`IO_NUM_n`** (bit mask for I/O #n) with **`IO_WRITE_SET`** or **`IO_WRITE_CLEAR`** to per-bit write helpers.

| Constant | Value | Use |
|----------|-------|-----|
| `IO_NUM_1` … `IO_NUM_32` | single-bit mask | pass directly as `IoMask_` |
| `IO_WRITE_SET` | 0 | OR bit on (`\|= IO_NUM_n`) |
| `IO_WRITE_CLEAR` | 1 | clear bit (`&= IO_MASK_ALL ^ IO_NUM_n`) |
| `IO_MASK_ALL` | `0xFFFFFFFF` | used internally for CLEAR |

Example: `uplIODInLogWrite(A_AXIS, IO_NUM_1, IO_WRITE_SET)`

Full-port value writes use `uplIODInLogWritePort` / `uplIODOutPortWritePort`.

### Integration example

```
uplIODOutModeConfig(A_AXIS, 1, DOUTMODE_USER_OUTPUT)
uplIODOutPortWritePort(A_AXIS, 0)
uplIODOutPortWrite(A_AXIS, IO_NUM_1, IO_WRITE_SET)
AWaitTime, 500
uplIODOutPortWrite(A_AXIS, IO_NUM_1, IO_WRITE_CLEAR)

l:din = uplIODInPortRead(A_AXIS)
if (uplIODInPortReadBit(A_AXIS, IO_NUM_1))
	uplIODInLogWrite(A_AXIS, IO_NUM_1, IO_WRITE_SET)
```

### Digital functions

Listed in source order: **Config → Read → Write** within digital inputs, then digital outputs.

**Digital inputs**

| Function | Returns | Maps to |
|----------|---------|---------|
| `uplIODInModeConfig(lAxis_, InputIndex_, Mode_)` | — | `PDInMode[InputIndex_]` |
| `uplIODInPortRead(lAxis_)` | `l` | `PDInPort` |
| `uplIODInPortReadBit(lAxis_, IoMask_)` | `l` | `PDInPort & IoMask_` (pass `IO_NUM_n`) |
| `uplIODInLogWritePort(lAxis_, value_)` | — | `PDInLog` (full 32-bit value) |
| `uplIODInLogWrite(lAxis_, IoMask_, Action_)` | — | SET: `\|= IoMask_`; CLEAR: `&= IO_MASK_ALL ^ IoMask_` |
| `uplIODInLogToggleBits(lAxis_, IoMask_)` | — | `PDInLog ^= IoMask_` |

**Digital outputs**

| Function | Returns | Maps to |
|----------|---------|---------|
| `uplIODOutModeConfig(lAxis_, OutputIndex_, Mode_)` | — | `PDOutMode[OutputIndex_]` |
| `uplIODOutSelectConfig(lAxis_, SelectIndex_, value_)` | — | `PDOutSelect[SelectIndex_]` |
| `uplIODOutTypeConfig(lAxis_, value_)` | — | `PDOutType` |
| `uplIODOutPortRead(lAxis_)` | `l` | `PDOutPort` |
| `uplIODOutLogRead(lAxis_)` | `l` | `PDOutLog` |
| `uplIODOutSelectRead(lAxis_, SelectIndex_)` | `l` | `PDOutSelect[SelectIndex_]` |
| `uplIODOutTypeRead(lAxis_)` | `l` | `PDOutType` |
| `uplIODOutPortWritePort(lAxis_, value_)` | — | `PDOutPort` (full 32-bit value) |
| `uplIODOutPortWrite(lAxis_, IoMask_, Action_)` | — | SET: `\|= IoMask_`; CLEAR: `&= IO_MASK_ALL ^ IoMask_` |
| `uplIODOutPortToggleBits(lAxis_, IoMask_)` | — | `PDOutPort ^= IoMask_` |

Mode constants: `DINMODE_*`, `DOUTMODE_*` in `uphGeneralConstants.puh2`.

### Analog functions

**Analog input** (Config → Read)

| Function | Returns | Maps to |
|----------|---------|---------|
| `uplIOAInModeSet(lAxis_, InputIndex_, Mode_)` | — | `PAInMode[InputIndex_]` |
| `uplIOAInPortRead(lAxis_, Index_)` | `f` | `PAInPort[Index_]`; **`Index_` > 0** |

**Analog output** — reserved section in `uplIO.pup2` (no functions yet).

### Constants (`uplIO.puh2` / `uphGeneralConstants.puh2`)

**`uplIO.puh2`**

- `IO_NUM_1` … `IO_NUM_32` — single-bit masks by I/O number
- `IO_WRITE_SET`, `IO_WRITE_CLEAR`, `IO_MASK_ALL`

**`uphGeneralConstants.puh2`**

- `DINMODE_*`, `DOUTMODE_*`, `DOUTTYPE_SINK` / `DOUTTYPE_SOURCE`
- `IO_DIN_COUNT` (32), `IO_DOUT_COUNT` (32), `IO_DOUT_SELECT_COUNT` (16)
- `DINPORT_DINn_BIT`, `DOUTPORT_DOUTn_BIT` / `_SET` / `_CLEAR` (legacy)
