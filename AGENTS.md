# AGENTS.md

Personal fork of [dbisu/pico-ducky](https://github.com/dbisu/pico-ducky): a USB Rubber
Ducky built on a Raspberry Pi Pico (RP2040) running CircuitPython.

## What this fork adds over upstream

Branch `fix/duckyscript3-await-and-if-continuation` carries two fixes for DuckyScript
3.0 execution in `duckyinpython.py`:

1. **Missing awaits (v4.0 regression)** — `parseLine()`, `runScript()` and
   `IF.runIf()` became async, but several internal call sites were not awaited,
   silently breaking `IF/ELSE`, `WHILE`, `FUNCTION`, `REPEAT` and `IMPORT`.
   Upstream issue #297; PR #354 covers only `REPEAT`.
2. **Continuation after `IF`/`END_IF`** — `runScript()` discarded the value returned
   by `parseLine()` and kept iterating the iterator that the `IF` block had already
   consumed with `list(codeIter)`, so everything after `END_IF` was silently skipped.
   Pre-existing since v3.2; found via hardware testing.

## Repo map

- `duckyinpython.py` — the DuckyScript interpreter (where the fixes live).
- `code.py`, `boot.py`, `pins.py` — bootstrap, USB storage mode and pin map
  (`GP0`=setup mode, `GP4/GP5/GP10/GP11`=payload select, `GP15`=USB storage toggle,
  `GP22`=re-run button).
- `webapp.py`, `wsgiserver.py` — web interface (Pico W only).
- `build_scripts/create_release_bundle.py` — release zip builder.
- `examples/` — sample payloads (`functions.dd`, `while_loops.dd`).

## Notes for future sessions

- The `upstream` remote is configured: sync with
  `git fetch upstream && git merge upstream/main`.
- Verification used for the fixes: `py_compile`, an AST audit (no async call left
  un-awaited) and a host-side simulator with CircuitPython stubs that runs
  `runScript()` against a sample payload. Hardware for end-to-end tests: 16 MB
  RP2040 "Pico v3" clone, CircuitPython 10.0.3, host Ubuntu GNOME (es+winkeys layout).
- License is GPL-2.0: keep `LICENSE` and copyright headers intact, and keep the
  change-notice comment lines at the top of modified files (GPLv2 §2a).
