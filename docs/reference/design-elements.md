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

Note: `Activity.getActivityInfo()` requires `Activity` permission. Prefer `ActivityMonitor.getInfo()` for HR when we already have that permission for steps.

#### Distance

```monkeyc
var info = ActivityMonitor.getInfo();
var distance = info.distance;  // Number (cm), null if no GPS fix
var distanceKm = (distance != null) ? (distance / 100000.0) : 0.0;
var distStr = Lang.format("$1$km", [distanceKm.format("%.1f")]);
```

#### Altitude

```monkeyc
// Requires Sensor permission in manifest.xml
using Toybox.Sensor;

var sensorInfo = Sensor.getInfo();
var altitude = sensorInfo.altitude;  // Number (m), null if no barometer data
var altStr = (altitude != null) ? Lang.format("$1$m", [altitude]) : "--m";
```

#### Ambient Pressure

```monkeyc
// Requires Sensor permission in manifest.xml
using Toybox.Sensor;

var sensorInfo = Sensor.getInfo();
var pressure = sensorInfo.pressure;  // Number (Pa), null if no sensor
// Convert Pa → hPa
var pressureHpa = (pressure != null) ? (pressure / 100.0).toNumber() : 0;
var pressureStr = Lang.format("$1$", [pressureHpa]);
```

**Trend arrow:** Compare current vs previous reading. If no previous reading available, show no arrow.

| Condition | Arrow |
|-----------|-------|
| Rising (> 1 hPa) | ↑ |
| Falling (< -1 hPa) | ↓ |
| Stable | → |

#### Active Calories

```monkeyc
var info = ActivityMonitor.getInfo();
var calories = info.calories;  // Number (kcal), null if no data
var calStr = (calories != null) ? calories.toString() : "--";
```

#### Beers Earned

```monkeyc
const CALORIES_PER_BEER = 150;  // ~150 kcal per beer
var info = ActivityMonitor.getInfo();
var calories = (info.calories != null) ? info.calories : 0;
var beers = calories / CALORIES_PER_BEER.toFloat();
var beerStr = Lang.format("$1$", [beers.format("%.1f")]);
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

### 8. Subscreen HR + Battery Ring

Instinct 2 Solar has a physical circular cutout (subscreen) at top-right (62×62 px). Display HR in it with a circular battery progress ring.

```monkeyc
var subscreen = WatchUi.getSubscreen();
// subscreen.x, subscreen.y, subscreen.width, subscreen.height

var centerX = subscreen.x + subscreen.width / 2;
var centerY = subscreen.y + subscreen.height / 2;
var radius  = subscreen.width / 2 - 2;  // 2px padding

var info = ActivityMonitor.getInfo();
var hr = (info.heartRate != null) ? info.heartRate : 0;
var battery = System.getSystemStats().battery;  // 0–100
```

**HR display:** Large digits centered in subscreen (`FONT_NUMBER_MEDIUM`). If HR is null, show `--`.

**Circular battery ring:**

```monkeyc
dc.setColor(Graphics.COLOR_WHITE, Graphics.COLOR_BLACK);
dc.setPenWidth(3);

// Background ring (full circle)
dc.drawCircle(centerX, centerY, radius);

// Progress arc
var angle = (battery / 100.0) * 360;  // degrees
// Draw arc from -90° (12 o'clock) clockwise
dc.drawArc(centerX, centerY, radius,
           Graphics.ARC_CLOCKWISE, 90, 90 + angle.toNumber());
```

Arc start at 90° = 12 o'clock position (Garmin coordinate system).

### 9. Sunline Divider + Moon Phase

Horizontal divider line below time with moon phase icon at center.

```monkeyc
var lineY = (dc.getHeight() * 0.72).toNumber();  // ~127
dc.setColor(Graphics.COLOR_WHITE, Graphics.COLOR_BLACK);
dc.setPenWidth(1);

// Left half
dc.drawLine(0, lineY, dc.getWidth() / 2 - 8, lineY);
// Right half
dc.drawLine(dc.getWidth() / 2 + 8, lineY, dc.getWidth(), lineY);

// Moon phase at center
var moonIcon = getMoonPhaseIcon(clockTime);  // returns drawable or char
```

**Moon phase calculation:**

```monkeyc
// Known new moon: Jan 29, 2025 (MJD 60676)
// Lunar cycle: 29.53 days
const NEW_MOON_MJD = 60676;
const LUNAR_CYCLE  = 29.53;

function getMoonPhase(now as Time.Moment) as Number {
    var mjd = Time.Gregorian.utcInfo(now, Time.FORMAT_SHORT).dayOfYear
              + (Time.Gregorian.utcInfo(now, Time.FORMAT_SHORT).year - 2000) * 365;
    var elapsed = now.value() - Gregorian.moment({:year=>2025, :month=>1, :day=>29}).value();
    var dayFraction = elapsed.toFloat() / 86400.0;  // seconds → days
    var phase = (dayFraction % LUNAR_CYCLE) / LUNAR_CYCLE;  // 0..1
    return phase;
}
```

**Moon phase icons:** Text-based fallbacks until PNG drawables exist:

| Phase range | Label | Unicode fallback |
|------------|-------|-----------------|
| 0.00–0.03, 0.97–1.00 | New Moon | `N` |
| 0.03–0.22 | Waxing Crescent | `(` |
| 0.22–0.28 | First Quarter | `D` |
| 0.28–0.47 | Waxing Gibbous | `G` |
| 0.47–0.53 | Full Moon | `O` |
| 0.53–0.72 | Waning Gibbous | `G` |
| 0.72–0.78 | Last Quarter | `D` |
| 0.78–0.97 | Waning Crescent | `)` |

### 10. Hash Marks (clock dial markers)

```monkeyc
for (var i = 0; i < 12; i += 1) {
    hashMarksArray[i][0] = (i / 12.0) * Math.PI * 2;
    hashMarksArray[i][1] = hm_factor * min_screen / 2;
}
```

### 11. Charts

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

## Fonts (12)

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

Base dimensions:

```monkeyc
var dcWidth  = dc.getWidth();   // 176
var dcHeight = dc.getHeight();  // 176
```

Subscreen-aware zones for current.txt layout:

```monkeyc
var ss = WatchUi.getSubscreen();
// ss.x, ss.y, ss.width, ss.height — subscreen at top-right

// Time center — shifted left to avoid subscreen
var timeCenterX = (dcWidth - ss.width) / 2;     // ~57
var timeCenterY = dcHeight * 0.55;               // ~97

// Date box — top-left corner
var dateBoxX = 4;
var dateBoxY = 4;
var dateBoxW = 48;
var dateBoxH = 32;

// Left data panel — fills area left of subscreen
var leftPanelX = 4;
var leftPanelY = ss.y + ss.height + 4;           // ~70
var leftPanelW = ss.x - 8;                       // ~106

// Sunline divider
var sunlineY    = dcHeight * 0.72;               // ~127
var moonPhaseX  = dcWidth / 2;
var moonPhaseY  = sunlineY;

// Bottom section
var bottomY      = sunlineY + 10;                 // ~137
var pressureX    = dcWidth * 0.25;                // ~44
var caloriesX    = dcWidth * 0.75;                // ~132

// Footer (beers earned)
var footerY = dcHeight - 12;                      // ~164

// Data labels and values — left panel spacing
var labelFont   = Graphics.FONT_XTINY;
var valueFont   = Graphics.FONT_TINY;
var rowHeight   = 16; // vertical spacing between data rows
```

---

## Data Sources

| Data            | API                                                  | Permissions     |
| -------------- | ---------------------------------------------------- | --------------- |
| Time           | `System.getClockTime()`                              | —               |
| Battery        | `System.getSystemStats().battery`                    | —               |
| Battery days   | `System.getSystemStats().batteryInDays`              | —               |
| Solar          | `System.getSystemStats().solarIntensity`             | —               |
| Steps          | `ActivityMonitor.getInfo().steps`                    | ActivityMonitor |
| Step Goal      | `ActivityMonitor.getInfo().stepGoal`                 | ActivityMonitor |
| Distance       | `ActivityMonitor.getInfo().distance` (cm)            | ActivityMonitor |
| Altitude       | `Sensor.getInfo().altitude` (m)                        | Sensor         |
| HR             | `ActivityMonitor.getInfo().heartRate` (bpm)          | ActivityMonitor |
| HR History     | `ActivityMonitor.getHeartRateHistory()`              | ActivityMonitor |
| Respiration    | `ActivityMonitor.getInfo().respirationRate`          | ActivityMonitor |
| SpO2           | `Activity.getActivityInfo().currentOxygenSaturation` | Activity        |
| Calories       | `ActivityMonitor.getInfo().calories` (kcal)          | ActivityMonitor |
| Pressure       | `Sensor.getInfo().pressure` (Pa)                     | Sensor         |
| Move Bar       | `ActivityMonitor.getInfo().moveBarLevel`             | ActivityMonitor |
| Active Minutes | `ActivityMonitor.getInfo().activeMinutesWeek`        | ActivityMonitor |
| Body Battery   | `SensorHistory.getBodyBatteryHistory()`              | SensorHistory   |
| Stress         | `SensorHistory.getStressHistory()`                   | SensorHistory   |
| BT Connected   | `System.getDeviceSettings().phoneConnected`          | Communications  |
| Notifications  | `System.getDeviceSettings().notificationCount`       | Communications  |
| Moon Phase     | Calculated from date                                 | —               |
| Subscreen      | `WatchUi.getSubscreen()`                             | —               |
| Weather        | `Weather.getCurrentConditions()`                     | Weather         |
| Date           | `Time.Gregorian.info()`                              | —               |