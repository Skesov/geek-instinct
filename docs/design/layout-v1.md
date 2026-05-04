# Design: geek-instinct Watch Faces

## Screen Layout (176×176, MIP 2-color)

```
┌─────────────────────┐
│                     │
│      12:34:56       │  ← Time center (FONT_NUMBER_HOT)
│                     │
│   HR 72   BAT 85%   |  ← HR left, battery right (FONT_SMALL)
│                     │
│     STEPS 5,432     │  ← Steps (FONT_SMALL)
│                     │
│  SOL 67%    MON 14  │  ← Solar left, date right (FONT_SMALL)
│                     │
└─────────────────────┘
```

**Note**: Icons (heart, sun, calendar) are not supported by system fonts. All labels use plain ASCII text:
- `HR` — heart rate
- `BAT` — battery
- `STEPS` — step count
- `SOL` — solar intensity
- Date as `DOW DD` (e.g., `MON 14`)

## Layout Grid

| Zone | Y range | Content |
|------|---------|---------|
| Top | 0–60 | Time (hh:mm:ss) |
| Middle-top | 60–100 | HR left, BAT right |
| Middle | 100–140 | Steps (centered) |
| Bottom | 140–176 | Solar left, date right |

Coordinates computed via `LayoutHelper` — no hardcoded pixels.

## View Variants

### MinimalView
- Time (hh:mm)
- Date (weekday + day)
- Battery %

### StandardView
- Time + seconds
- Heart rate (HR)
- Steps
- Battery %
- Body battery (optional, via SensorHistory)

### SolarView
- Everything in StandardView
- Solar intensity bar
- Analog clock hands (hour + minute)
- Body battery + stress (SensorHistory)

## Color Usage (MIP 2-color)

| Purpose | Color |
|---------|-------|
| Background | `COLOR_BLACK` |
| Primary text/lines | `COLOR_WHITE` |

Only `COLOR_WHITE` and `COLOR_BLACK` render correctly on Instinct 2 Solar (MIP 2-color).

## Icons

All icons are 24×24 or 32×32 PNG drawables placed in `resources/drawables/`. System fonts do not support Unicode glyphs beyond basic ASCII.