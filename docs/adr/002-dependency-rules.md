# ADR 002: Toybox Isolation Rule

## Status

Accepted

## Context

All device I/O in Connect IQ lives in `Toybox.*` namespaces. Without a rule, Toybox calls spread across every file — making the code tied to hardware and harder to reason about.

## Decision

**One file imports Toybox; all other source files are pure business logic.**

```
Toybox.* APIs ──► DataSource.mc ──► (all other source files)
                          ▲
                          │
               only place that calls Toybox
```

- `DataSource.mc` imports all `Toybox.*` namespaces and exposes data via plain methods.
- `Views/`, `Utils/`, `App.mc` — no Toybox imports, no hardware calls.
- If Toybox API changes, only `DataSource.mc` needs updating.

## Consequences

### Pros
- `DataSource` methods are callable on the simulator without a view.
- Swapping a data source (e.g., for a mock) requires changing one file.
- Clear ownership: all hardware coupling lives in `DataSource`.

### Cons
- Extra indirection — for a single-view watch face, direct Toybox calls in the View may be simpler.
- Monkey C has no interface keyword — "interfaces" are conventions (method names only).

## Clarification

This is not "Dependencies Inward" from Clean Architecture. Watch faces have no domain layer — only data collection and rendering. The rule is simply **Toybox isolation**, not a layered architecture.

## References

Toybox API docs: `references/api_reference.md`