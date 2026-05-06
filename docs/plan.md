# Plan: geek-instinct Watch Face Development

## Overview

Single custom watch face for Garmin Instinct 2 Solar. Data-rich layout replicating the design from `workbench/scratch/current.txt` (ID 96317).

## Target Device

| Parameter | Value |
|-----------|-------|
| Device | Instinct 2 Solar |
| Product ID | `instinct2` |
| Resolution | 176 × 176 px |
| Display | MIP 2-color (B&W only) |
| API Level | 3.4.0 |

## Architecture

See `docs/adr/001-project-structure.md` for details.

```
source/
├── App.mc           # Entry point
├── View.mc           # Watch face — all drawing and data access
├── Layout.mc         # Coordinate formulas
└── Config.mc         # Constants
```

## Feature Set (from current.txt)

| # | Feature | Data Source | Zone |
|---|---------|-------------|------|
| 1 | Time HH:MM (leading zero) | `System.getClockTime()` | Center |
| 2 | Date (e.g., Tue 05) with border | `Gregorian.info()` | Top-left |
| 3 | Steps with count | `ActivityMonitor.getInfo().steps` | Top-left |
| 4 | Distance (km) | `ActivityMonitor.getInfo().distance` (cm → km) | Left panel |
| 5 | Altitude (m) | `Sensor.getInfo().altitude` (requires `Sensor` perm) | Left panel |
| 6 | Battery % + days remaining | `System.getSystemStats()` | Left panel |
| 7 | Bluetooth + notification icons | `System.getDeviceSettings()` | Left panel |
| 8 | Heart Rate in subscreen | `ActivityMonitor.getInfo().heartRate` | Top-right cutout (62×62) |
| 9 | Circular battery ring around HR | `System.getSystemStats().battery` | Top-right cutout |
| 10 | Sunline divider + moon phase | Calculated from date | Below time |
| 11 | Ambient pressure (hPa) + trend | `Sensor.getInfo().pressure` (Pa → hPa) | Bottom-left |
| 12 | Active calories + icon | `ActivityMonitor.getInfo().calories` | Bottom-right |
| 13 | Beers earned | `calories / 150` | Footer |

## Phases

### Phase 1: Project Setup & "Hello Watch"

| # | Task | Status |
|---|------|--------|
| 1.1 | Create `manifest.xml` with permissions | pending |
| 1.2 | Create `monkey.jungle` | pending |
| 1.3 | Create `App.mc` entry point | pending |
| 1.4 | Create `View.mc` — time + date only | pending |
| 1.5 | Build and run on simulator | pending |

### Phase 2: Full Data Implementation

| # | Task | Status |
|---|------|--------|
| 2.1 | Implement subscreen HR + battery ring | pending |
| 2.2 | Implement date box + steps | pending |
| 2.3 | Implement left panel (dist, alt, batt, BT) | pending |
| 2.4 | Implement sunline + moon phase | pending |
| 2.5 | Implement bottom section (pressure, calories) | pending |
| 2.6 | Implement beers earned | pending |

### Phase 3: Polish

- `onPartialUpdate()` for seconds
- Icons (heart, footprint, flame, beer, moon phase)
- Real device testing
- Final `.iq` export

## Permissions Required

```xml
<iq:uses-permission id="ActivityMonitor"/>
<iq:uses-permission id="UserProfile"/>
<iq:uses-permission id="Communications"/>
<iq:uses-permission id="Sensor"/>
```
