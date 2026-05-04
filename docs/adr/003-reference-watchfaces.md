# ADR 003: Reference Watchface Selection

## Status

Accepted

## Context

We need open-source implementation references to build our watchface from scratch.

## Decision

Use two reference watchfaces:

| Reference | Purpose | Why |
|-----------|---------|-----|
| **geektime** (rishubil/geektime) | Data-heavy layout | Targets `instinct2` directly; shows HR, steps, breath, SpO2, battery-as-days |
| **ElegantAnalog-Watchface** (bhugh/) | Analog hands + solar | Shows working analog hands, solar clock, inset circle patterns |

## Consequences

### Pros
- Both repos target Instinct 2 / 176×176 — screen dimensions match exactly.
- geektime shows a full data-heavy layout we can adapt for our StandardView/SolarView.
- ElegantAnalog shows the analog hand drawing math we need for SolarView.

### Cons
- Both repos are without explicit license — treat as study reference only.
- SmartArcs is GPL-3.0 — copyleft license limits reuse; excluded as primary reference.

## Reversal Condition

If neither reference is sufficient once we start implementation, search for additional repos targeting `instinct2s_solar` specifically.