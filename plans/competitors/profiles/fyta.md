# FYTA

**Category:** Sensor + hub + app + cloud + B2B platform. **The most direct competitor.**
**Source:** user-supplied research from FYTA's official site (20 Oct 2026), copied into this profile. Prices are the EU site prices and are not landed prices in India.

## What it is

Not just a sensor maker. FYTA has consumer sensors, a hub, a plant-care app, cloud data, a B2B dashboard, an open API, smart-home integrations and a reseller program.

```text
FYTA sensors --BLE--> FYTA Wi-Fi Hub --Wi-Fi--> FYTA Cloud --> App / API / B2B dashboard
```

- **Without a hub:** sensor data is readable only while the phone is within Bluetooth range.
- **With a hub:** cloud data and notifications work even when the phone is away.
- **Other connectivity:** Terra and an LTE-M hub for suitable use cases.

**This is essentially our planned architecture** (many sensor nodes → one hub → cloud → app), already productized.

## Hardware and prices (EU site)

| Product | Measures | Specs | Price |
|---|---|---|---|
| **Mini** | Soil moisture (VWC), EC | BLE 5.0, CR2450/LIR2450 replaceable, up to 12 months battery, IP64, interchangeable probe lengths | from €29.99 |
| **Beam G2** | Moisture, EC, light (PPFD/DLI), temperature | BLE 5.0, CR2450/LIR2450, about 9–12 months battery, IP64, interchangeable probe lengths | from €39.99 |
| **Sphere** | Light (PPFD/DLI), temperature, air humidity | Environmental sensor, not a soil sensor | €44.99 |
| **Terra** | Robust outdoor sensor/hub option | | from €79.99 |
| **Wi-Fi Hub** | Connectivity | | €39.99 |
| **Grow Set** | 3× Mini + 1× Sphere + 1× Hub | 3 plants + shared environment | €199.99 |
| **Grove Set** | 3× Mini + 2× Beam G2 + 1× Sphere + 1× Hub | Plant collections | €249.99 |

**Design principle:** specialised sensors rather than every sensor in every device. Per-plant soil sensors plus one shared room sensor. This matches Botanical Legacy's plant-level vs room-level split and our own plan.

## App

- Real-time readings (moisture, EC/nutrients, light, temperature, humidity).
- Watering notifications and species-specific tips.
- **4,800+ plant profiles.**
- The earlier India notes say the app is free with no hidden subscription. Re-verify.

## B2B, API and integrations

- **Science & Industry / B2B:** interior landscaping, facility services, professional plant care, automated irrigation, vertical farming, universities and research.
- **Web dashboard:** visualise, monitor many plants, trends, data export.
- **FYTA Beam API:** plant-status data, API docs, dashboard access, IFTTT. Discord developer community.
- **Commercial caveat:** the API is free only for non-commercial and non-institutional use. Building a commercial product on FYTA's API needs a licensing or partnership deal.
- **Integrations:** Home Assistant, Homey, IFTTT, API.
- **Reseller program:** modular product, cross-sell/upsell, smart-home enthusiasts, less after-sales effort because the app guides care.

## Cost vs our target

| Item | FYTA (EU price) | Our target | Gap |
|---|---|---|---|
| Soil node | Mini €29.99 | ₹250–₹400 | Roughly 8–10× at about ₹100/€ (assumed rate; check) |
| Hub | €39.99 | about ₹500 | Roughly 8× |
| 3 plants + room + hub | Grow Set €199.99 (about €67 per plant) | not yet costed | |

India landed cost also includes international shipping, import duty/tax, currency conversion and payment costs. Compute it for a real comparison.

## Threat assessment (from the research)

| Area | Threat |
|---|---|
| Consumer plant app | Medium |
| Physical sensors | Very high |
| Multi-sensor architecture | Very high |
| Hub architecture | Very high |
| Plant intelligence | High |
| B2B | Very high |
| API / integrations | Very high |
| India-specific affordability | Opportunity |
| Very-low-cost DIY hardware | Opportunity |

## What not to compete on

Their research concludes we should not position on: "an app that tells you when to water", "multiple sensors on a hub", "sensor data + plant intelligence", or "an API for plant sensor data".

## Remaining opportunity

**India-first economics:** a low-cost node plus one inexpensive hub for many plants, with IoT complexity hidden from the user (no BLE, 433 MHz, Home Assistant, MQTT or ESPHome knowledge needed). The open question is whether sensing at a fraction of FYTA's cost is still accurate enough for better plant-care decisions.

## Gaps / to verify

- The source is a single summary of FYTA's site. Link-level citations were not provided, so see [sources.md](../sources.md#fyta) for the pages named.
- Accuracy, calibration method, sampling rate and range are not covered. Needed for the sensor deep dive.
- The "433 MHz" mention in the source appears without context (our own radio option?).
- Whether FYTA's plant intelligence uses history/anomaly detection or only species thresholds is unclear (rated "High" without detail).
