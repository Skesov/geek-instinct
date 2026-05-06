# ADR 002: Data Access Pattern

## Status

Accepted (supersedes previous version — 2026-05-05)

## Context

The previous ADR proposed strict Toybox isolation: only `DataSource.mc` imports Toybox, all other files are "pure logic". This rule is incompatible with Monkey C reality:

- **Views MUST import `Toybox.WatchUi`** — `WatchUi.WatchFace` is the base class, `Graphics.Dc` is the draw context. There is no way to render pixels without touching Toybox.
- **Monkey C has no interfaces** — "abstraction" is naming convention only. A `DataSource` class adds indirection without compile-time guarantees.
- **Watch faces are data collection + rendering** — there is no domain layer to protect. Separating data reads from rendering just adds call overhead in a resource-constrained runtime.

The strict isolation rule was written by an LLM unfamiliar with Monkey C constraints. It is rescinded.

## Decision

**Views import Toybox namespaces directly. No artificial abstraction layer.**

```monkeyc
using Toybox.WatchUi;
using Toybox.Graphics;
using Toybox.System;
using Toybox.ActivityMonitor;
using Toybox.Time;
using Toybox.Time.Gregorian;

class MyView extends WatchUi.WatchFace {
    function onUpdate(dc as Graphics.Dc) as Void {
        // Read directly, no intermediary
        var clock = System.getClockTime();
        var info  = ActivityMonitor.getInfo();
        // ... draw ...
    }
}
```

Rules that remain:

- **One read per sensor per frame.** If two draw functions need steps, read `ActivityMonitor.getInfo()` once into a local var, not twice.
- **No Toybox calls in `Layout.mc` or `Config.mc`.** These utility modules are pure coordinate/constant files — keep them import-free.
- **Class-level vars for mutable cache.** Allocate `mClock`, `mInfo`, `mStats` as class fields, not locals — avoids GC pressure from repeated allocation in `onUpdate()`.

## Consequences

### Pros
- Code matches Monkey C reality. No fighting the framework.
- Data flow is visible: `onUpdate()` → read → draw. One file to trace.
- Fewer files, fewer imports, simpler compile graph.

### Cons
- `View.mc` will be the only file importing Toybox — it will have 6–8 `using` statements. Acceptable for a single-file watch face.
- If we later need unit tests, mocking Toybox is impossible anyway (Monkey C has no DI). Testing remains device-only.

## Rationale

> "Don't abstract what you can't change." Monkey C provides no mechanism to swap Toybox implementations. An abstraction layer over Toybox is indirection without benefit.

## References

- Connect IQ API docs: Toybox namespaces are the SDK — there is no "layer below" them.
- Garmin sample watch faces: all draw data directly in `onUpdate()` with no abstraction.
