# ADR 001: Project Structure

## Status

Accepted

## Context

We need to develop multiple watch face variants for Garmin Instinct 2 Solar with shared code and a clear growth path. The team is a single developer working locally.

## Decision

Mono-repository with one `manifest.xml`, one `DataSource`, and multiple View classes.

```
source/
├── App.mc                    # entry point — returns the selected View
├── DataSource.mc            # all Toybox.* calls — single integration point
├── Views/
│   ├── BaseWatchFaceView.mc # shared: layout, draw helpers, view lifecycle
│   ├── MinimalView.mc       # P0: time + date + battery
│   ├── StandardView.mc      # P1: + HR + steps
│   └── SolarView.mc         # P2: + solar + analog hands
└── Utils/
    ├── LayoutHelper.mc      # coordinate formulas, no hardcoded pixels
    └── Colors.mc             # COLOR_WHITE / COLOR_BLACK constants
```

- `DataSource` owns all Toybox.* imports — no other file calls hardware APIs.
- `BaseWatchFaceView` provides layout grid, draw primitives, and the `onUpdate()` lifecycle.
- View variants inherit from `BaseWatchFaceView` and override only `drawContent()` to add or change data fields.
- `LayoutHelper` uses `dc.getWidth()` / `dc.getHeight()` — no hardcoded coordinates.

## Consequences

### Pros
- All variants share one codebase and one build artifact.
- Adding a new variant = copy a View class → override `drawContent()`.
- All device-specific logic flows through `DataSource` — one place to change.

### Cons
- Each watchface variant is a separate `.prg` entry point in `manifest.xml`. The user picks one via the Garmin watch face picker — not by switching inside a running app.
- Shared base class adds indirection — for a single variant, simpler to have one flat file.

## Rationale

> "Start Simple" — we are one developer. The indirection cost is acceptable because we will have 3+ variants and want DRY reuse of the data-access layer.

## Reversal Condition

If we need independent deployment cadences or per-variant release channels, split into separate `manifest.xml` projects.