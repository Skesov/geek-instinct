# ADR 001: Project Structure

## Status

Accepted (supersedes previous version — 2026-05-05)

## Context

We develop a single custom watch face for Garmin Instinct 2 Solar (`instinct2`). The previous ADR described a multi-variant approach (MinimalView, StandardView, SolarView) — this was rejected in favour of a single data-rich watch face replicating an existing design (workbench/scratch/current.txt).

The watch face is a single screen — no pages, no gesture switching, no multiple view variants.

## Decision

**Single watch face with one View class.** Flat structure with utility modules.

```
source/
├── App.mc                    # entry point — returns the watch face view
├── View.mc                   # watch face lifecycle, data reading, drawing
├── Layout.mc                 # coordinate formulas, zone definitions
└── Config.mc                 # constants (thresholds, colors, fonts)
```

- `App.mc` — one-line entry point.
- `View.mc` — extends `WatchUi.WatchFace`. Reads sensors directly, draws all zones.
- `Layout.mc` — pure coordinate functions. Input: dc dimensions. Output: zone rectangles. No Toybox imports.
- `Config.mc` — compile-time constants: `.moonPhaseBaseDate`, `.caloriesPerBeer`, etc.

No `BaseWatchFaceView`, no `DataSource` abstraction, no view inheritance tree. For a single watch face, a flat structure is simpler and avoids premature abstraction.

## Consequences

### Pros
- One file handles everything — easy to trace data flow.
- No indirection layers. Sensor reads happen where they're rendered.
- Adding a new data field = add one draw function in View.mc, update Layout.mc.

### Cons
- View.mc may grow large (~400–500 lines). Mitigated by extracting coordinate logic to Layout.mc and constants to Config.mc.
- If a second watch face variant is needed later, refactoring will be required. KISS: we have one watch face today.

## Rationale

> "Start Simple." We are one developer with one watch face design. A multi-variant architecture solves a problem we don't have.

## Reversal Condition

If we need multiple watch face variants with independent feature sets, extract a base class from View.mc and reintroduce the inheritance hierarchy from the original ADR-001.
