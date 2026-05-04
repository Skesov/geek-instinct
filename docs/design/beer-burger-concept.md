# Concept: Earned Treats (Beer & Burgers)

Motivation concept — "earned food/drink" through burned calories.

## Formula

```monkeyc
items = calories_burned / calories_per_item
```

## Data Source

```monkeyc
var info = ActivityMonitor.getInfo();
var calories = info.calories;  // kcal burned today
```

## Reference Values

### Beer (0.5L / ~17oz)

| Type             | kcal     | Units per 1000 kcal |
| ---------------- | -------- | ------------------- |
| Regular (5% ABV) | ~150     | ~6.7                |
| Light            | ~100     | 10                  |
| Strong (8%)      | ~200     | 5                   |
| **Average**      | **~150** | **~6.7**            |

### Burgers

| Burger          | kcal | Units per 1000 kcal |
| --------------- | ---- | ------------------- |
| Big Mac         | ~550 | ~1.8                |
| Quarter Pounder | ~530 | ~1.9                |
| McChicken       | ~400 | 2.5                 |
| Cheeseburger    | ~300 | ~3.3                |
| Regular Burger  | ~250 | 4                   |

### Reference Burger

Using **300 kcal** (Cheeseburger average) for simplicity — **~3.3 per 1000 kcal**

## Examples

Assuming 1200 kcal burned today:

| Item         | Calculation | Result              |
| ------------ | ----------- | ------------------- |
| Beer (0.5L)  | 1200 / 150  | **8 beers** (4L!)   |
| Big Mac      | 1200 / 550  | **2.2 burgers**     |
| Cheeseburger | 1200 / 300  | **4 cheeseburgers** |

## UI Concept

```text
┌─────────────────────────┐
│         12:34           │
│                         │
│    🍺 5.2  🍔 1.8       │
│                         │
│   HR 72    STEPS 5,432  │
│                         │
│  [solar bar]  [date]    │
└─────────────────────────┘
```

Shorter format:

```text
BEER 5  BURGER 2
```

## Display Rules

- **Beer**: fractional ok (`5.2`) or integer (`5`)
- **Burger**: fractional ok, but often rounded to integer
- **Zero**: hide or show `0`

## Open Questions

1. Threshold: show only if > 0.5 units?
2. Format: `5.2` vs `5` vs `5.3L`?
3. Which burger as reference — Big Mac or average (~300)?
4. Add other treats (ice cream, donuts)?
