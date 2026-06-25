## UPL_LibraryExamples

Clean Agito/UPL project for running **examples and tests** against the shared library modules.

### Layout

| File | Role |
|------|------|
| `UPL_LibraryExamples.pup2` | Main program — uncomment one module section or test block at a time |
| `UPL_LibraryExamples.puh2` | Project-only `#define` / `#definevar` |
| `UPL_LibraryExamples.puj2` | Agito project file (source list, IDE settings) |
| `uplUtility.*`, `uplMotion.*`, `uplCNC.*`, `uplIO.*` | Library modules — **maintained here** (this folder holds the latest of every module) |
| `uphGeneralConstants.puh2` | Shared constants (maintained here) |
| `README_upl*.md`, `LIBRARY_CONVENTIONS.md`, `README_IDEplus.md` | Per-module API reference, library conventions, and the IDE+ language guide (maintained here) |

**Develop all library code here.** `CNCMotions_specifications` is a separate consumer project, and **all shared library sources and docs are hard-linked** into it (edits propagate automatically): every `upl*.pup2` / `upl*.puh2`, `uphGeneralConstants.puh2`, `LIBRARY_CONVENTIONS.md`, and `README_uplIO.md` / `README_uplUtility.md` / `README_uplCNC.md` / `README_Integration_Example.md`.

**The one exception is `README_uplMotion.md`** — it is a plain copy (not linked), so edit both copies to keep them in sync (they are currently identical). `CHANGELOG.md` exists only in `CNCMotions_specifications`; `README.md` and `README_IDEplus.md` exist only here.

### How to use

1. Open `UPL_LibraryExamples.puj2` in the Agito IDE.
2. In `UPL_LibraryExamples.pup2`, uncomment **one** block to run:
   - Utility, Motion, CNC, or I/O examples, **or**
   - the **Tests** block (e.g. `testDInReadBit`)
3. Comment out other active blocks (including `AProgHaltThis` if you need the program to continue).
4. Build and run on the drive.

Start with **Utility + Motion** before CNC or I/O tests. See `README_Integration_Example.md` for a minimal CNC flow.

### Library source of truth

**`UPL_LibraryExamples` is the source of truth** for all `upl*` modules, `uphGeneralConstants.puh2`, and the docs — develop and version here (git `develop` / `master`, pushed to GitHub). `CNCMotions_specifications` consumes the library (partly via hard links). The sibling scratch projects under `../` (e.g. `blink_io`, `for_def_tests`, `assignment_test2`) are early experiments/tests — **ignore them** for development; they're only useful as references for IDE+ language behavior (reflected in `README_IDEplus.md`).
