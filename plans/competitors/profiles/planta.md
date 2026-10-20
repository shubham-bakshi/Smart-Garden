# Planta

**Category:** Software-first plant-care management.
**Scale (company claim):** 7M+ users, 30M+ plants managed.
**Physical sensor:** None. No continuous soil sensor.

## Features

- **Add plants** by search or photo ID, assigned to a site (e.g. Living Room).
- **Watering algorithm:** 30+ parameters plus local weather and climate. Inputs include site, distance from window, soil type and pot size, so two Monsteras can get different schedules.
- **Feedback loop:** the user can snooze ("soil still wet") or water early, and future schedules adjust.
- **Light Meter** (premium): measures room light and compares it with the plant's needs.
- **Dr. Planta:** photo diagnosis (yellow leaves, pests, weak growth) with a treatment plan. AI diagnosis is premium.
- **Other reminders:** fertilizing, misting, repotting, cleaning, plus custom tasks (rotate, move, pesticide, seasonal).
- **Journal:** plants, locations and photos, so it works as a plant-management system.
- **Care guides:** per-plant and per-problem content (humidity, grow lights, soil, pests, propagation, etc.).

## Takeaways for us

- Planta already has strong personalization, so that is not our differentiator.
- Its gap is how it learns the soil state: it **asks the user** ("is the soil still wet?"). A sensor would answer that automatically.
- Light is not "more is better". Required light is species-specific, so any light sensing must be compared against the plant's requirements.
- Worked example of the product we're aiming for:

```text
Moisture 32% + watered 2 days ago + Monstera + low light this week + normally dries in 5 days
→ "Don't water yet. Soil is drying slower than normal; light is lower. Move it closer to the window."
```

Sources: see [sources.md](../sources.md#planta).
