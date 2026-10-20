# Botanical Legacy

**Category:** Software + model layer on top of third-party sensors. **Closest conceptual match to our idea.**
**Physical sensor:** No proprietary sensor. Integrates third-party sensors through **Home Assistant**.

## Core concept: Digital Shadow

A continuously updated per-plant model of the plant's state that predicts what happens next.

```text
Species + pot volume + substrate + light + watering history
+ observed dry-down + environment/season
→ individual plant model → projected soil moisture → predicted watering date
```

The next watering date comes from projected moisture crossing a **plant-specific threshold**, not a fixed interval. This means "individual-plant learning" is not unique to us.

## Sensor integration

- **Architecture:** sensor → Home Assistant (internet-accessible) → Botanical Legacy → Digital Shadow → forecast and alerts.
- **Measurements:** soil moisture, temperature, humidity, illuminance, conductivity.
- **Compatible examples:** Xiaomi Mi Flora / Flower Care (BLE), Zigbee soil probes, ESP32 + capacitive sensor via ESPHome.
- **Binding:** a sensor can be bound to a specific plant or to a room/zone.
- **Behavior:**
  - polled about every 10 minutes
  - history kept for charts
  - a fresh moisture reading becomes the current state and the forecast continues from it
  - room temperature/humidity influence the model
  - conductivity is tracked as a nutrition/salt trend
  - offline sensors trigger alerts

## Ideas worth borrowing

1. **Model + measurement:** the model estimates state without a sensor, and a real reading overrides it. The sensor changes the model; it is not just a number on screen.
2. **Dry-down learning** per plant/pot.
3. **Automatic watering detection** from a large moisture jump (e.g. 25% → 58%), which reduces manual logging.
4. **Room-level sensors:** one temp/humidity sensor serves many plants. Plant-level: moisture, EC, light. Room-level: temperature, humidity. This is the natural basis for our hub.
5. **Sensor-failure fallback:** failure should reduce confidence, not break the app.
6. **What-if simulation:** water, move to brighter light, feed or mist, and see the projected result.

## Weaknesses / opportunities

- Home Assistant is powerful but intimidating for normal plant owners. A **plug-and-play hub** (sensor → hub → app) is an opening.
- Total hardware cost varies with the third-party sensors chosen. A dedicated low-cost kit may beat it per plant.
- Compatible does not mean equal quality. Accuracy, repeatability, drift, battery life and soil-specific behavior are unknown.
- Availability: web plus Android. iOS is via web/PWA.

**Deeper detail** (override rules, thresholds, sensor quality, pricing, competitor claims): [botanical-legacy-sensor-research.md](../botanical-legacy-sensor-research.md).

**Pricing:** sensors need a paid plan (Cultivator, about €6.99/month, up to 50 plants). The free Observer plan has 5 plants and no sensors. They sell no hardware.

Sources: see [sources.md](../sources.md#botanical-legacy).
