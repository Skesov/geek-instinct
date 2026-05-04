# Watch Face Design Reference

For Garmin Instinct 2 / Instinct 2 Solar (176×176, MIP 2-color, API 3.4.0).

## Sources

| Reference     | URL                                      | Features                              |
| ------------- | ---------------------------------------- | ------------------------------------- |
| geektime      | github.com/rishubil/geektime             | Data-heavy, HR/steps/breath/SpO2 graphs |
| ElegantAnalog | github.com/bhugh/ElegantAnalog-Watchface | Analog hands, Solar Clock             |

---

## Screen Zones (176×176)

```
┌─────────────────────────────────────┐
│  [status icons: DND, conn, notif]    │  ← top row (y: 0–20)
│                                     │
│                                     │  ← upper middle (y: 20–70)
│         [analog clock or            │
│          large time display]        │
│                                     │
│                                     │  ← center (y: 70–100)
│  [data: HR, steps, breath, etc]     │  ← middle (y: 100–140)
│                                     │
│  [solar bar] [date] [battery]       │  ← bottom row (y: 140–176)
└─────────────────────────────────────┘
```

---

## Elements

### 1. Time

#### Digital Formats

| Format          | Example     | Font              | Notes                  |
| --------------- | ---------- | ----------------- | ---------------------- |
| `hh:mm`         | `09:41`    | `FONT_NUMBER_HOT` | Large, centered        |
| `hh:mm:ss`      | `09:41:23` | `FONT_NUMBER_HOT` | Seconds smaller        |
| `hh:mm` + `AM/PM` | `09:41 AM` | `FONT_NUMBER_HOT` | 12h mode              |

#### Analog Hands

Implementation: `drawHand()` with polygon coordinates via `generateHandCoordinates()`.

**Hand types** (ElegantAnalog):

| type | Description                   | Appearance               |
| ---- | ----------------------------- | ------------------------ |
| 0    | Big Pointer                   | Filled rectangle         |
| 1    | Needle                        | Thin line                |
| 2    | Triangle                      | Standard triangle        |
| 3    | Outline Blunt                 | Rectangle outline        |
| 4    | Blanked Rectangle Outline     | Outline with gap         |
| 5    | Triangle Outline              | Triangle outline         |
| 6    | Blanked Triangle Outline      | Outline with gap         |

**Hand positions:**

- Main dial center: `width/2, height/2`
- Inset circle: `centerX_circle, centerY_circle, radius_circle`

**Angle formula** (0° = 12 o'clock, clockwise):

```monkeyc
var hourHandAngle = (((clockTime.hour % 12) * 60) + clockTime.min);
hourHandAngle = hourHandAngle / (12 * 60.0) * Math.PI * 2;

var minuteHandAngle = (clockTime.min / 60.0) * Math.PI * 2;

var secondHand = (clockTime.sec / 60.0) * Math.PI * 2;
```

**Hand lengths** (relative to screen):

```monkeyc
hourHandLength   = height / 6;    // ~29px on 176 screen
minuteHandLength = height / 3;     // ~58px on 176 screen
secondHandLength = width * 0.475; // ~83px on 176 screen
```

### 2. Status Icons

| Icon            | State        | Drawable                   |
| -------------- | --------------- | ---------------------------- |
| Do Not Disturb | on/off        | `DndOnIcon` / `DndOffIcon`   |
| Connection     | on/off        | `ConnOnIcon` / `ConnOffIcon` |
| Notification   | has/none      | `NotiOnIcon` / `NotiOffIcon` |
| Battery        | charging/normal | `BattOnIcon` / `BattOffIcon` |

Icon sizes: 24×24 or 32×32 PNG in `resources/drawables/`.

### 3. Activity Data

#### Heart Rate

```monkeyc
var activityInfo = Activity.getActivityInfo();
var hr = activityInfo.currentHeartRate;  // Number (bpm)

// Graph: line with min/max for last 20 minutes
var hrIterator = ActivityMonitor.getHeartRateHistory(HR_GRAPH_POINT_COUNT, false);
var hrMax = hrIterator.getMax();
var hrMin = hrIterator.getMin();
```

#### Steps

```monkeyc
var info = ActivityMonitor.getInfo();
var steps = info.steps;         // Number
var stepGoal = info.stepGoal;   // Number (default 1500)

// Graph: bar chart of step deltas
```

#### Respiration Rate (Breath)

```monkeyc
var respirationRate = activityMonitorInfo.respirationRate;  // Number
```

#### SpO2 (Oxygen Saturation)

```monkeyc
var currentOxygenSaturation = activityInfo.currentOxygenSaturation;  // Number (%)
```

### 4. Battery

```monkeyc
var stats = System.getSystemStats();
var battery = stats.battery;           // Float 0–100

// batteryInDays — only on some devices
var batteryInDays = 100;
if (stats has :batteryInDays && stats.batteryInDays != null) {
    batteryInDays = stats.batteryInDays;
}
```

**Display variants:**

- Percentage: `85%`
- Days: `12D` (if `batteryInDays` available)
- Charging: show percentage instead of days

### 5. Solar Intensity

```monkeyc
var stats = System.getSystemStats();
var solar = stats.solarIntensity;  // Number 0–100 or null!

// ALWAYS null-check before arithmetic
if (solar != null) {
    // draw solar bar
}
```

**Solar bar** — horizontal strip at bottom of screen:

```monkeyc
var barMax = (screenWidth * 0.34).toNumber();  // ~60px
var barX   = (screenWidth - barMax) / 2;
var barY   = (dc.getHeight() * 0.85).toNumber();
var barW   = (solar * barMax / 100).toNumber();

dc.setColor(Graphics.COLOR_WHITE, Graphics.COLOR_BLACK);
dc.drawRectangle(barX, barY, barMax, 6);
if (barW > 0) {
    dc.fillRectangle(barX, barY, barW, 6);
}
```

### 6. Date

```monkeyc
var now = Time.now();
var info = Gregorian.info(now, Time.FORMAT_SHORT);

var dateStr = Lang.format("$1$ $2$ $3$", [
    info.day_of_week,  // "MON"
    info.month,        // "JAN"
    info.day          // "14"
]);
```

**Formats:**

- `MON 14` — day of week + day of month (short)
- `JAN 14` — month + day
- `2024-01-14` — full date

### 7. Move Bar / Activity Minutes

```monkeyc
var moveBarLevel = info.moveBarLevel;  // 0–5 (5 = goal reached)
var moveExpired = (moveBarLevel >= 5);

// Active minutes
var activeMinutesWeek = info.activeMinutesWeek.total;
var activeMinutesWeekGoal = info.activeMinutesWeekGoal;
var activeMinutesDay = info.activeMinutesDay.total;
```

**Visualization:** blocks `████░░░░` or `+` marks.

### 8. Inset Circle (Solar Clock)

ElegantAnalog uses inset circle for:

- Sunrise/Sunset time
- Dawn/Dusk markers
- Second hand in miniature

```monkeyc
var ss = WatchUi.getSubscreen();
// ss.x, ss.y, ss.width, ss.height

var radius_circle = ss.height / 2 + 1;
var centerX_circle = ss.x + radius_circle;
var centerY_circle = ss.y + radius_circle;
```

### 9. Hash Marks (clock dial markers)

```monkeyc
for (var i = 0; i < 12; i += 1) {
    hashMarksArray[i][0] = (i / 12.0) * Math.PI * 2;
    hashMarksArray[i][1] = hm_factor * min_screen / 2;
}
```

### 10. Charts

#### Line Chart (HR, Breath)

```monkeyc
function drawLineChart(dc, maxValue, minValue, values, options) as Void {
    var chartX = options.get("chartX");
    var chartY = options.get("chartY");
    var pointMaxHeight = options.get("pointMaxHeight");
    var pointGap = options.get("pointGap");

    for (var i = 0; i < values.size(); i++) {
        var value = values[values.size() - i - 1];
        if (value == null) { continue; }

        var height = (((value - minValue).toFloat() / (maxValue - minValue))
                      * pointMaxHeight).toNumber();
        var x = chartX - i * pointGap;
        var y = chartY + pointMaxHeight - height;

        if (lastX != null && lastY != null) {
            dc.drawLine(lastX, lastY, x, y);
        }
        lastX = x;
        lastY = y;
    }
}
```

#### Bar Chart (Steps)

```monkeyc
function drawBarChart(dc, maxValue, values, chartX, chartY, barMaxHeight, barWidth, barGap) as Void {
    for (var i = 0; i < values.size(); i++) {
        var value = values[values.size() - i - 1];
        if (value == null) { continue; }

        var height = ((value.toFloat() / maxValue) * barMaxHeight).toNumber();
        var x = chartX - i * (barWidth + barGap);
        var y = chartY + barMaxHeight - height;

        dc.fillRectangle(x, y, barWidth, height);
    }
}
```

---

## Fonts

| Constant           | Description      | Availability |
| ----------------- | -------------- | ----------- |
| `FONT_NUMBER_HOT` | Large digits   | Always      |
| `FONT_LARGE`       | Medium         | Always      |
| `FONT_MEDIUM`      | Medium-small   | Always      |
| `FONT_SMALL`       | Small          | Always      |
| `FONT_TINY`        | Very small     | Always      |

**Custom fonts:** Monkey C does not support custom TTF in production. Use system fonts and ASCII-compatible characters.

---

## Color Palette (MIP 2-color)

| Constant                 | Actual color | Notes          |
| ---------------------- | ---------------- | ----------- |
| `Graphics.COLOR_WHITE` | White (light)    | Use this     |
| `Graphics.COLOR_BLACK` | Black (dark)    | Use this     |

**All other colors** (`COLOR_RED`, `COLOR_GREEN`, etc.) **may render incorrectly** — mapping to WHITE or BLACK depends on device. Do not use.

---

## Battery Optimization

### onPartialUpdate()

Called every second instead of full `onUpdate()`. Use `dc.setClip()` to limit redraw area:

```monkeyc
function onPartialUpdate(dc as Graphics.Dc) as Void {
    dc.setClip(SEC_X, SEC_Y, SEC_W, SEC_H);
    dc.setColor(Graphics.COLOR_BLACK, Graphics.COLOR_BLACK);
    dc.clear();
    dc.setColor(Graphics.COLOR_WHITE, Graphics.COLOR_BLACK);
    dc.drawText(SEC_X, SEC_Y, font, secString, Graphics.TEXT_JUSTIFY_LEFT);
    dc.clearClip();
}
```

### Buffering

```monkeyc
_offscreenBuffer = Graphics.createBufferedBitmap({
    :width => dc.getWidth(),
    :height => dc.getHeight()
});
```

### Second Hand

- **Long hand** on main dial: uses more battery
- **Short hand** (center or inset circle): uses less battery
- **Disable** second hand: maximum battery saving

---

## Layout Helper (formulas)

```monkeyc
var width  = dc.getWidth();   // 176
var height = dc.getHeight();  // 176
var centerX = width / 2;      // 88
var centerY = height / 2;      // 88
var minScreen = width < height ? width : height;  // 176

// Zone positions
var topRowY    = height * 0.05;   // ~9
var centerY    = height * 0.45;   // ~79
var middleY    = height * 0.60;    // ~106
var bottomY    = height * 0.85;   // ~150
```

---

## Data Sources

| Data            | API                                                  | Permissions     |
| -------------- | ---------------------------------------------------- | --------------- |
| Time           | `System.getClockTime()`                              | —               |
| Battery        | `System.getSystemStats().battery`                    | —               |
| Solar          | `System.getSystemStats().solarIntensity`             | —               |
| Steps          | `ActivityMonitor.getInfo().steps`                    | ActivityMonitor |
| HR             | `ActivityMonitor.getInfo().heartRate`                | ActivityMonitor |
| HR History     | `ActivityMonitor.getHeartRateHistory()`              | ActivityMonitor |
| Respiration    | `ActivityMonitor.getInfo().respirationRate`          | ActivityMonitor |
| SpO2           | `Activity.getActivityInfo().currentOxygenSaturation` | Activity        |
| Move Bar       | `ActivityMonitor.getInfo().moveBarLevel`             | ActivityMonitor |
| Active Minutes | `ActivityMonitor.getInfo().activeMinutesWeek`        | ActivityMonitor |
| Body Battery   | `SensorHistory.getBodyBatteryHistory()`              | SensorHistory   |
| Stress         | `SensorHistory.getStressHistory()`                   | SensorHistory   |
| Weather        | `Weather.getCurrentConditions()`                     | Weather         |
| Date           | `Time.Gregorian.info()`                              | —               |