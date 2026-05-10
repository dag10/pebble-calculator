# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Pebble Calculator is a touchscreen calculator app for the Pebble smartwatch (Emery platform only). It supports both standard and RPN (Reverse Polish Notation) modes with a 4-register stack (T/Z/Y/X). Written in C using the Pebble SDK v3.

## Build Commands

```bash
pebble build          # Build the app
pebble install        # Install to connected watch or emulator
pebble build && pebble install --emulator emery   # Build and run in Emery emulator
pebble logs           # View app logs
```

The build system uses `waf` (Python-based) configured in `wscript`. All C sources under `src/c/` are compiled automatically via glob.

## Architecture

The app is split into focused modules under `src/c/`:

- **calculator.c** — App entry point, lifecycle, touch/button input handling, persistent storage, and Clay config (AppMessage). This is the glue that wires everything together.
- **calc_engine** — Core calculator logic. Manages the entry buffer, standard-mode pending operator, and RPN 4-register stack. All button presses are translated to `CalcAction` enums and processed by `calc_engine_handle_action()`.
- **calc_format** — Number formatting and parsing. Pebble has no `strtod` or `math.h`, so this module implements custom double-to-string and string-to-double conversion, including scientific notation fallback for large/small values.
- **calc_buttons** — Button layout definitions and hit-testing. Defines a 5x4 grid (200x228px Emery screen). Each button has separate labels/actions for standard vs RPN mode.
- **calc_ui** — Rendering. Draws the display area (X register, secondary line for Y/pending-op) and the button grid with press states.
- **calc_fonts** — Font loading and access (LECO for numbers, Gothic for labels).
- **calc_icons** — Procedural drawing of operator icons (+, -, x, /, =, backspace) using Pebble graphics primitives.

**Settings** are managed via [Rebble Clay](https://github.com/nicewithgreat/clay) (`src/pkjs/`). Config options (RPN mode, haptic feedback, keep backlight) are sent to the watch via AppMessage and persisted with `persist_write_*`.

## Key Constraints

- **No math.h / no strtod**: `calc_format.c` and `calc_engine.c` implement their own floating-point helpers. Be careful modifying number formatting — there's no standard library fallback.
- **Emery-only**: The app targets the 200x228px Emery display with touchscreen. All layout constants in `calc_buttons.h` are hardcoded for this screen size.
- **Display width**: Numbers are capped at 13 digit characters (`CALC_FORMAT_MAX_DIGITS`). The minus sign consumes a digit slot; the decimal point does not.
