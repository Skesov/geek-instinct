# Layout: geek-instinct Watch Face (v2)

Based on `workbench/scratch/current.txt` (ID 96317). Target: 176×176, MIP 2-color, Instinct 2 Solar.

## Screen Zones

```
┌─────────────────────────────────────┐ y=0
│  ┌───────┐               ┌───────┐  │
│  │ TUE   │  STEPS  94   │  ♥    │  │ ← Top section + subscreen (HR)
│  │  05   │               │  47   │  │
│  └───────┘               └───────┘  │ y=64
│  DIST 0.1km   ALT 76m              │ ← Left data cluster
│                                     │
│  BAT 93.4%  26d  ●BT  ●NOTIF       │ ← Battery + status icons
│                                     │
│             00:45                   │ ← Primary time (center-ish)
│                 :23                 │    Seconds on right side
│  ────────── 🌑 ──────────          │ ← Sunline divider + moon phase
│                                     │
│  1001 ↑            2 🔥            │ ← Pressure (left) / Calories (right)
│                                     │
│              0.0 🍺                 │ ← Beers earned (footer)
└─────────────────────────────────────┘ y=176
```

## Zone Details

### Zone 1: Top Section (y: 0–62)

Subdivided into left panel and right subscreen.

| Sub-zone | Position | Content |
|----------|----------|---------|
| Date box | Top-left, x≈4, y≈4 | Day-of-week + day-of-month in rectangular border |
| Steps | Below date, left side | Footprint icon + step count |
| Distance | Left panel, below steps | `0.1km` (converted from cm) |
| Altitude | Left panel, next to distance | `76m` |
| Battery | Left panel, below altitude | `93.4% 26d` (percentage + estimated days) |
| BT icon | Left panel, bottom | Bluetooth connected indicator |
| Notif icon | Left panel, bottom | Notification count indicator |

### Zone 2: Subscreen HR (top-right circular cutout)

Rect obtained from `WatchUi.getSubscreen()`. On Instinct 2 Solar: 62×62 px at top-right.

| Element | Position | Content |
|---------|----------|---------|
| Heart icon | Above HR value | Small heart drawable or "♥" text |
| HR value | Center of subscreen | Current HR in large digits (FONT_NUMBER_MEDIUM) |
| Battery ring | Circumference of subscreen | Circular progress bar mapped to battery % |

If HR is null (no sensor data), show `--` instead.

### Zone 3: Primary Time (y: 62–115)

| Element | Position | Content |
|---------|----------|---------|
| Time | x≈85, y≈95 | HH:MM with leading zero (FONT_NUMBER_HOT) |
| Seconds | Right of time, y≈95 | `:23` smaller font (FONT_NUMBER_MEDIUM) |

Time is shifted slightly left to avoid the subscreen area. Exact position: center of the area below subscreen.

### Zone 4: Sunline Divider (y: 115–125)

| Element | Position | Content |
|---------|----------|---------|
| Horizontal line | Full width | Thin line spanning display |
| Moon phase icon | Center of line | Moon phase icon for current date |

Moon phase calculated from date. Cycle length: ~29.53 days. Ref: known new moon date as epoch.

### Zone 5: Bottom Section (y: 125–165)

| Sub-zone | Position | Content |
|----------|----------|---------|
| Pressure | Bottom-left | Ambient pressure in hPa (e.g., `1001`) + vertical trend arrow (↑↓→) |
| Calories | Bottom-right | Active calories (e.g., `2`) + flame icon |

### Zone 6: Footer (y: 165–176)

| Element | Position | Content |
|---------|----------|---------|
| Beers earned | Bottom center | `0.0 🍺` — calories / 150 (beer mug icon) |

## Font Selection

| Element | Font | Reason |
|---------|------|--------|
| Time HH:MM | `FONT_NUMBER_HOT` | Largest, most legible for primary content |
| Seconds | `FONT_NUMBER_MEDIUM` | Smaller than time, still numeric clarity |
| HR value | `FONT_NUMBER_MEDIUM` | Large enough in small subscreen area |
| Data labels (DIST, ALT, BAT) | `FONT_XTINY` | Minimal, non-intrusive labels |
| Data values (0.1km, 76m, 93.4%) | `FONT_TINY` | Compact but legible |
| Date | `FONT_TINY` | Fits in small box |

## Coordinate Philosophy

All coordinates calculated by `Layout.mc` functions. Input: `dc.getWidth()`, `dc.getHeight()`, `WatchUi.getSubscreen()`. Output: rectangles and text positions.

No hardcoded pixel values in `View.mc`. Zone boundaries expressed as fractions of screen dimensions.

## Rendering Order

1. Clear screen (`COLOR_BLACK`)
2. Draw date box border (`COLOR_WHITE` rectangle outline)
3. Draw date text
4. Draw left panel data (distance, altitude, battery)
5. Draw status icons (BT, notifications)
6. Draw subscreen HR + battery ring
7. Draw time HH:MM + seconds
8. Draw sunline divider
9. Draw moon phase icon
10. Draw pressure + calories
11. Draw beers earned footer
