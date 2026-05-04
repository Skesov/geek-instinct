# Plan: geek-instinct Watch Face Development

## Overview

Development of custom watch faces for Garmin Instinct 2 / Instinct 2 Solar via Connect IQ SDK and open-source reference implementations.

## Target Device

| Parameter | Value |
|-----------|-------|
| Device | Instinct 2 Solar |
| Product ID | `instinct2s_solar` |
| Resolution | 176 × 176 px |
| Display | MIP 2-color (B&W only) |
| API Level | 3.4.0 |

---

## Phases

### Phase 1: Reference & Study

| # | Task | Details | Status |
|---|------|---------|--------|
| 1.1 | Study open-source watchfaces | Analyze geektime, ElegantAnalog for patterns | pending |
| 1.2 | Identify feature set | Define what our watchface will display | pending |
| 1.3 | Design layout | Create layout sketch for 176×176 display | pending |

---

### Phase 2: Base Project Setup

| # | Task | Details | Status |
|---|------|---------|--------|
| 2.1 | Install Connect IQ SDK | SDK Manager + VS Code Monkey C extension | pending |
| 2.2 | Create monorepo structure | As defined in `docs/adr/001-project-structure.md` | pending |
| 2.3 | Verify build pipeline | "Hello Watch" — time + date only | pending |
| 2.4 | Run on simulator | Target `instinct2s_solar` | pending |

---

### Phase 3: Feature Implementation

Priority | Feature | Data Source | Status |
----------|---------|-------------|--------|
| P0 | Time (hh:mm:ss) | `System.getClockTime()` | pending |
| P0 | Date | `System.getClockTime()` | pending |
| P0 | Battery % | `System.getSystemStats().battery` | pending |
| P1 | Heart Rate (HR) | `ActivityMonitor.getInfo().heartRate` | pending |
| P1 | Steps | `ActivityMonitor.getInfo().steps` | pending |
| P1 | Solar intensity | `System.getSystemStats().solarIntensity` | pending |
| P2 | Analog hands | `Graphics.drawLine()` | pending |
| P2 | Indicator icons | `drawable/*.png` | pending |
| P3 | Body Battery | `SensorHistory.getBodyBatteryHistory()` | pending |
| P3 | Stress level | `SensorHistory.getStressHistory()` | pending |

---

### Phase 4: Polish

- `onPartialUpdate()` optimization for battery saving
- Real device testing
- Final `.prg` / `.iq` export

---

## Project Structure

```
geek-instinct/
├── manifest.xml
├── monkey.jungle
├── source/
│   ├── App.mc
│   ├── Views/
│   │   ├── BaseWatchFaceView.mc
│   │   ├── MinimalView.mc
│   │   ├── StandardView.mc
│   │   └── SolarView.mc
│   └── Utils/
│       ├── DataSource.mc
│       ├── LayoutHelper.mc
│       └── Colors.mc
├── resources/
│   ├── drawables/
│   └── layouts/
└── docs/
    ├── adr/
    ├── reference/
    └── design/
```

---

## Key Technical Decisions

| Decision | Rationale |
|----------|-----------|
| Mono-repo | Shared base class, multiple watchface variants |
| BaseWatchFaceView | Centralized layout + data logic — DRY |
| MIP 2-color only | Instinct 2 Solar supports `COLOR_WHITE` / `COLOR_BLACK` only |
| API Level 3.4 | Minimum for Instinct 2 Solar compatibility |