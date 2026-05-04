# Reference Watchfaces

Open-source implementations used as study material for building geek-instinct.

## Reference 1: geektime

- **URL**: https://github.com/rishubil/geektime
- **Stars**: 12
- **Target**: `instinct2` (176×176, MIP 2-color)
- **Last updated**: Nov 2025

**Patterns to study**: Data-heavy layout with HR graphs, steps, breath rate, SpO2, weather, UTC offset, battery-as-days.

---

## Reference 2: ElegantAnalog-Watchface

- **URL**: https://github.com/bhugh/ElegantAnalog-Watchface
- **Stars**: 5
- **Target**: `instinct2`, `instinct2s`, `instinct2x`, and more
- **Last updated**: Jan 2025

**Patterns to study**: Analog clock hands (`drawLine()`), solar clock / dawn-dusk, inset circle, battery options.

---

## Implementation Patterns

### WatchFace entry point (Toybox.WatchFace)

```monkeyc
using Toybox.WatchUi;
using Toybox.Graphics;
using Toybox.System;

class MyWatchFaceView extends WatchUi.WatchFace {

    var mWidth as Number = 0;
    var mHeight as Number = 0;

    function initialize() {
        WatchUi.WatchFace.initialize();
    }

    function onLayout(dc as Graphics.Dc) as Void {
        mWidth  = dc.getWidth();   // 176 on Instinct 2
        mHeight = dc.getHeight();  // 176 on Instinct 2
    }

    function onUpdate(dc as Graphics.Dc) as Void {
        dc.setColor(Graphics.COLOR_WHITE, Graphics.COLOR_BLACK);
        dc.clear();

        var time = System.getClockTime();
        var timeStr = Lang.format("$1$:$2$", [time.hour, time.min.format("%02d")]);
        dc.drawText(mWidth / 2, mHeight / 2, Graphics.FONT_NUMBER_HOT, timeStr,
                    Graphics.TEXT_JUSTIFY_CENTER | Graphics.TEXT_JUSTIFY_VCENTER);
    }
}
```

### Data sources

| Data | API |
|------|-----|
| Time | `System.getClockTime()` → `.hour`, `.min`, `.sec` |
| Battery | `System.getSystemStats().battery` → Float 0–100 |
| Solar | `System.getSystemStats().solarIntensity` → Number or `null` |
| Steps | `ActivityMonitor.getInfo().steps` → Number |
| Heart Rate | `ActivityMonitor.getInfo().heartRate` → Number |
| Body Battery | `SensorHistory.getBodyBatteryHistory(opts)` → Iterator |
| Stress | `SensorHistory.getStressHistory(opts)` → Iterator |