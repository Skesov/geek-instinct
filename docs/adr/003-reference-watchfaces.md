# ADR 003: Reference Watchfaces

## Status

Accepted (updated 2026-05-05)

## Context

We need open-source implementation references to build our watchface.

## Decision

Use three references — one primary design target, two implementation pattern sources.

### Primary Design Target

| Reference | Purpose |
|-----------|---------|
| **current.txt watch face** (ID 96317) | Full layout specification. Data-rich single screen: subscreen HR, time center, activity stats, sunline divider, moon phase, pressure, calories, beers earned. |

This is the watch face currently installed on the developer's device. It serves as the exact layout specification — we aim to replicate it approximately (not pixel-perfect).

### Implementation Pattern Sources

| Reference | URL | Purpose |
|-----------|-----|---------|
| **geektime** | github.com/rishubil/geektime | Data-heavy layout, battery-as-days, step formatting, UTC offset. Targets `instinct2` directly. |
| **ElegantAnalog** | github.com/bhugh/ElegantAnalog-Watchface | Subscreen handling (`WatchUi.getSubscreen()`), analog hands, circular progress bars, solar clock. |

### Patterns taken from each

| Pattern | Source | Used for |
|---------|--------|----------|
| `WatchUi.getSubscreen()` | ElegantAnalog | Positioning HR in top-right circular cutout |
| Circular progress bar | ElegantAnalog | Battery ring around HR |
| Battery in days | geektime | `batteryInDays` with fallback |
| Step count formatting | geektime | Compact representation (5.4K) |

## Consequences

### Pros
- Primary target is a known-good design (used daily by the developer).
- Reference repos cover the technical unknowns: subscreen layout, circular drawing.
- All three target Instinct 2 / 176×176.

### Cons
- current.txt is a textual description only, not source code — layout interpretation requires experimentation.
- geektime and ElegantAnalog are without explicit license — study reference only.

## Reversal Condition

If subscreen coordinates or circular drawing cannot be replicated from ElegantAnalog, search for additional `instinct2` repos with subscreen usage.
