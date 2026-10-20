# Greg

**Category:** Software-first plant care + community. App Store tagline: "zero-guesswork plant care app & community".
**Physical sensor:** None found in the App Store listing or public product information.

## Features

- **PlantVision** identifies the species and estimates pot size, distance to the nearest window and window orientation from a photo.
- **Precision Watering** gives a custom plan (when and how much) from species, plant size and home environment.
- **Learning loop:** the developer says the schedule is calibrated from species, pot, light and local weather, and learns from inputs such as snoozing a reminder or marking a plant as watered early.
- **Paid "Super Greg":** Indoor Light Gauge and Seasonal Updates (care adapts as local weather changes).
- **Other care:** fertilizing, plant progress and life tracking.
- **Community:** Q&A, experienced growers, shared content. This is a major pillar.

## How it works (inferred from public descriptions)

```text
Plant profile → user-provided/inferred environment → weather + season
              → user feedback (watered / snoozed) → software model → when + how much
```

The exact algorithm is proprietary.

## Takeaways for us

- Greg already covers what we first thought would differentiate us: species + environment + history → personalized care.
- Greg depends on the user keeping state accurate. If the user forgets to log a watering, the model is wrong. Sensors could detect watering automatically (a hypothesis to validate).
- Our potential edge is **measured** conditions rather than **predicted** ones:
  - Greg: "Low light, large pot, recent watering → probably don't water yet."
  - Us: "Moisture 48%, 1,200 lux, drying 35% slower than usual → don't water, and here's why."
- This is a potential advantage, not a proven one.

Sources: see [sources.md](../sources.md#greg).
