## UPL_LibraryExamples

Clean Agito/UPL project for running **examples and tests** against the shared library modules.

### Layout

| File | Role |
|------|------|
| `UPL_LibraryExamples.pup2` | Main program — uncomment one module section or test block at a time |
| `UPL_LibraryExamples.puh2` | Project-only `#define` / `#definevar` |
| `UPL_LibraryExamples.puj2` | Agito project file (source list, IDE settings) |
| `uplUtility.*`, `uplMotion.*`, `uplCNC.*`, `uplIO.*` | Library modules (copies of `../CNCMotions_specifications`; `uplIO.*` is hard-linked) |
| `uphGeneralConstants.puh2` | Shared constants (copy; keep in sync with specifications folder) |
| `README_upl*.md`, `LIBRARY_CONVENTIONS.md` | Per-module API reference and library conventions (copies; keep in sync) |

Develop library code in **`CNCMotions_specifications`**, then sync copies here. Only `uplIO.puh2` / `uplIO.pup2` are hard-linked today — other module files must be copied after edits.

### How to use

1. Open `UPL_LibraryExamples.puj2` in the Agito IDE.
2. In `UPL_LibraryExamples.pup2`, uncomment **one** block to run:
   - Utility, Motion, CNC, or I/O examples, **or**
   - the **Tests** block (e.g. `testDInReadBit`)
3. Comment out other active blocks (including `AProgHaltThis` if you need the program to continue).
4. Build and run on the drive.

Start with **Utility + Motion** before CNC or I/O tests. See `README_Integration_Example.md` for a minimal CNC flow.

### Library source of truth

Develop and version library code in **`CNCMotions_specifications`**. This project is a thin shell for experiments — no duplicate module copies to maintain.
