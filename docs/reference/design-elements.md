# Watch Face Design Reference

Для Garmin Instinct 2 / Instinct 2 Solar (176×176, MIP 2-color, API 3.4.0).

## Источники

| Референс      | URL                                      | Особенности                               |
| ------------- | ---------------------------------------- | ----------------------------------------- |
| geektime      | github.com/rishubil/geektime             | Data-heavy, графики HR/шагов/дыхания/SpO2 |
| ElegantAnalog | github.com/bhugh/ElegantAnalog-Watchface | Аналоговые стрелки, Solar Clock           |

---

## Зоны экрана (176×176)

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

## Элементы

### 1. Время

#### Цифровые форматы

| Формат            | Пример     | Шрифт             | Notes                |
| ----------------- | ---------- | ----------------- | -------------------- |
| `hh:mm`           | `09:41`    | `FONT_NUMBER_HOT` | Большой, центральный |
| `hh:mm:ss`        | `09:41:23` | `FONT_NUMBER_HOT` | Секунды мелко        |
| `hh:mm` + `AM/PM` | `09:41 AM` | `FONT_NUMBER_HOT` | 12h mode             |

#### Аналоговые стрелки

Реализация: `drawHand()` с polygon coordinates через `generateHandCoordinates()`.

**Типы стрелок** (ElegantAnalog):

| type | Описание                  | Внешний вид               |
| ---- | ------------------------- | ------------------------- |
| 0    | Big Pointer               | Заполненный прямоугольник |
| 1    | Needle                    | Тонкая линия              |
| 2    | Triangle                  | Треугольник (обычный)     |
| 3    | Outline Blunt             | Контур прямоугольника     |
| 4    | Blanked Rectangle Outline | Контур с промежутком      |
| 5    | Triangle Outline          | Контур треугольника       |
| 6    | Blanked Triangle Outline  | Контур с промежутком      |

**Позиции стрелок:**

- Центр главного циферблата: `width/2, height/2`
- Inset circle: `centerX_circle, centerY_circle, radius_circle`

**Формула угла** (0° = 12 часов, по часовой стрелке):

```monkeyc
var hourHandAngle = (((clockTime.hour % 12) * 60) + clockTime.min);
hourHandAngle = hourHandAngle / (12 * 60.0) * Math.PI * 2;

var minuteHandAngle = (clockTime.min / 60.0) * Math.PI * 2;

var secondHand = (clockTime.sec / 60.0) * Math.PI * 2;
```

**Длина стрелок** (относительно экрана):

```monkeyc
hourHandLength   = height / 6;    // ~29px на 176
minuteHandLength = height / 3;     // ~58px на 176
secondHandLength = width * 0.475; // ~83px на 176
```

### 2. Статусные иконки

| Иконка         | Состояние       | Drawable                     |
| -------------- | --------------- | ---------------------------- |
| Do Not Disturb | вкл/выкл        | `DndOnIcon` / `DndOffIcon`   |
| Connection     | вкл/выкл        | `ConnOnIcon` / `ConnOffIcon` |
| Notification   | есть/нет        | `NotiOnIcon` / `NotiOffIcon` |
| Battery        | зарядка/обычная | `BattOnIcon` / `BattOffIcon` |

Размер иконок: 24×24 или 32×32 PNG в `resources/drawables/`.

### 3. Данные активности

#### Heart Rate

```monkeyc
var activityInfo = Activity.getActivityInfo();
var hr = activityInfo.currentHeartRate;  // Number (bpm)

// График: линия с min/max за последние 20 мин
var hrIterator = ActivityMonitor.getHeartRateHistory(HR_GRAPH_POINT_COUNT, false);
var hrMax = hrIterator.getMax();
var hrMin = hrIterator.getMin();
```

#### Steps

```monkeyc
var info = ActivityMonitor.getInfo();
var steps = info.steps;         // Number
var stepGoal = info.stepGoal;   // Number (default 1500)

// График: bar chart дельты шагов
```

#### Respiration Rate (Breath)

```monkeyc
var respirationRate = activityMonitorInfo.respirationRate;  // Number
```

#### SpO2 ( Oxygen Saturation)

```monkeyc
var currentOxygenSaturation = activityInfo.currentOxygenSaturation;  // Number (%)
```

### 4. Battery

```monkeyc
var stats = System.getSystemStats();
var battery = stats.battery;           // Float 0–100

// batteryInDays — только на некоторых устройствах
var batteryInDays = 100;
if (stats has :batteryInDays && stats.batteryInDays != null) {
    batteryInDays = stats.batteryInDays;
}
```

**Варианты отображения:**

- Проценты: `85%`
- Дни: `12D` (если есть `batteryInDays`)
- При зарядке: показывать проценты вместо дней

### 5. Solar Intensity

```monkeyc
var stats = System.getSystemStats();
var solar = stats.solarIntensity;  // Number 0–100 или null!

// ВСЕГДА проверяй на null
if (solar != null) {
    // рисовать solar bar
}
```

**Solar bar** — горизонтальная полоса внизу экрана:

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

**Форматы:**

- `MON 14` — день недели + день месяца (краткий)
- `JAN 14` — месяц + день
- `2024-01-14` — полная дата

### 7. Move Bar / Activity Minutes

```monkeyc
var moveBarLevel = info.moveBarLevel;  // 0–5 (5 = goal reached)
var moveExpired = (moveBarLevel >= 5);

// Active minutes
var activeMinutesWeek = info.activeMinutesWeek.total;
var activeMinutesWeekGoal = info.activeMinutesWeekGoal;
var activeMinutesDay = info.activeMinutesDay.total;
```

**Визуализация:** три поля `████░░░░` или `+` значки.

### 8. Inset Circle (Solar Clock)

ElegantAnalog использует inset circle для:

- Sunrise/Sunset время
- Dawn/Dusk маркеры
- Second hand в миниатюрном виде

```monkeyc
var ss = WatchUi.getSubscreen();
// ss.x, ss.y, ss.width, ss.height

var radius_circle = ss.height / 2 + 1;
var centerX_circle = ss.x + radius_circle;
var centerY_circle = ss.y + radius_circle;
```

### 9. Hash Marks (метки на циферблате)

```monkeyc
for (var i = 0; i < 12; i += 1) {
    hashMarksArray[i][0] = (i / 12.0) * Math.PI * 2;
    hashMarksArray[i][1] = hm_factor * min_screen / 2;
}
```

### 10. Графики

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

## Шрифты

| Константа         | Описание       | Доступность |
| ----------------- | -------------- | ----------- |
| `FONT_NUMBER_HOT` | Большие цифры  | Всегда      |
| `FONT_LARGE`      | Средний        | Всегда      |
| `FONT_MEDIUM`     | Средний-мелкий | Всегда      |
| `FONT_SMALL`      | Мелкий         | Всегда      |
| `FONT_TINY`       | Очень мелкий   | Всегда      |

**Кастомные шрифты:** Monkey C не поддерживает кастомные TTF в production. Используй системные шрифты и ASCII-совместимые символы.

---

## Цветовая палитра (MIP 2-color)

| Константа              | Фактический цвет | Notes       |
| ---------------------- | ---------------- | ----------- |
| `Graphics.COLOR_WHITE` | Белый (светлый)  | Только этот |
| `Graphics.COLOR_BLACK` | Чёрный (тёмный)  | Только этот |

**Все остальные цвета** (`COLOR_RED`, `COLOR_GREEN`, etc.) **могут отображаться некорректно** — маппинг на WHITE или BLACK зависит от устройства. Не используй.

---

## Оптимизация батареи

### onPartialUpdate()

Вызывается каждую секунду вместо полного `onUpdate()`. Используй `dc.setClip()` для ограничения области перерисовки:

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

- **Long hand** на главном циферблате: ~83px, потребляет больше battery
- **Short hand** (в центре или inset circle): ~40px, экономит ~50%
- **Disable** second hand: максимальная экономия

---

## Layout Helper (формулы)

```monkeyc
var width  = dc.getWidth();   // 176
var height = dc.getHeight();  // 176
var centerX = width / 2;      // 88
var centerY = height / 2;      // 88
var minScreen = width < height ? width : height;  // 176

// Позиции зон
var topRowY    = height * 0.05;   // ~9
var centerY    = height * 0.45;   // ~79
var middleY    = height * 0.60;    // ~106
var bottomY    = height * 0.85;   // ~150
```

---

## Data Sources

| Data           | API                                                  | Permissions     |
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
