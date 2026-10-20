# Differentiation & Validation Plan

## Thesis

> Build an **affordable, plug-and-play physical sensing network** for indoor plants, then use continuous measurements to improve **plant-specific care decisions**.

The hardware is not the product. The product is:

```text
Sensor → reliable measurement → plant state → historical context
       → plant-specific model → anomaly detection → actionable recommendation
```

Target: ₹250–₹400 per plant node and about ₹500 for the hub, to be confirmed with a BOM. FYTA's EU prices (Mini €29.99, hub €39.99, Grow Set €199.99) are the benchmark; see [profiles/fyta.md](profiles/fyta.md).

**Architecture is not a differentiator:** FYTA already ships plant nodes + shared room sensor + hub + cloud. Our edge has to be cost and simplicity.

For every sensor, ask: *what better decision can this measurement help us make?*

## Where we could still differentiate

1. **Hardware cost** (₹250–₹400 per node, if reliable).
2. **One hub for many plants** (economical for 10–20 plants).
3. **Plug-and-play** (buy kit → plug in hub → open app → add plant → place sensor), with no Home Assistant.
4. **Calibration** for common indoor substrates instead of arbitrary sensors.
5. **Decision intelligence:** "3 plants need attention today", not 20 charts.
6. **Anomaly detection with explanation.** Example: soil still wet after 8 days when it normally dries in 5, with light 30% lower. The app investigates light, temperature, humidity, drainage, compaction and root condition.
7. **Explainable recommendations:** "Don't water yet. The soil is wetter than this plant's usual threshold, and it is drying slower because light has been lower this week."
8. **Automatic state tracking** (watering detected from a moisture jump), which removes manual logging.

## Proposed architecture

```text
Plant node (moisture + light + optional temp/humidity, low-power MCU)
      ↓ BLE or Zigbee
ESP32 hub → Wi-Fi → backend → plant-care engine → app
```

- **Plant-level:** soil moisture, EC, light.
- **Room-level:** temperature, humidity.
- **Wireless:** BLE or Zigbee for battery nodes. Wi-Fi draws too much power for tiny battery nodes.

| Protocol | Pros | Cons |
|---|---|---|
| BLE | Low power, cheap, widely supported | Range, multi-room needs testing |
| Zigbee | Low power, mesh, commercial ecosystem | Needs coordinator, more complexity |
| Wi-Fi | Simple, direct | High power, poor fit for batteries |

## Sensor requirements

Absolute accuracy is not required. These matter more:

- consistency and repeatability
- useful relative changes
- long-term stability
- low power
- reliable wireless
- correlation with care decisions

Decision accuracy matters more than absolute measurement accuracy. A reliable plant-specific dry threshold can make an uncalibrated sensor useful.

## Validation sequence

1. **Sensor validation:** capacitive moisture, light, temp/humidity, and commercial wireless sensors. Test controlled moisture states (very dry → saturated), different substrates (potting mix, cocopeat, soil + compost, perlite mix), repeatability across units, and drift/corrosion/battery/connectivity over weeks to months.
2. **One node:** ESP32 + capacitive moisture + light sensor + battery + BLE.
3. **3–5 nodes:** multi-node communication, range, battery, hub, data reliability.
4. **Real plants:** 5–10 species. Record soil, pot, light, watering, moisture, temperature and humidity.
5. **Compare models:** software-only vs sensor-assisted.
6. **Build the product** only after step 5 shows sensors improve decisions.

## The key experiment

About 10 plants, several weeks, two systems:

- **A — software-only:** species, pot, room, light, weather, watering history.
- **B — sensor-assisted:** A plus actual moisture, light, temperature and humidity.

Measure watering-prediction error, false alerts, missed watering, overwatering and user interventions.

**If B isn't materially better, don't build the IoT layer. If it is, continue.**

Also test specifically:

- Can we know the actual soil state rather than predict it?
- Can we measure light continuously rather than rely on user-entered window info?
- Can we auto-detect watering events?
- Can we learn each plant/pot/soil drying curve?
- Can we detect anomalies (wet too long, drying too fast, sudden or persistent low light)?

## Next research task

Physical-sensing deep dive on **Botanical Legacy sensors, LetPlant/SDESA and FYTA**, comparing:

sensor technology · moisture method · light method · temp/humidity · conductivity · accuracy claims · calibration · sampling frequency · battery · wireless protocol · range · hub architecture · sensors per hub · hardware price · cost per plant · India availability · long-term reliability

Then build and test our own ₹250–₹400 node against the strongest alternatives.
