# Greg — Competitor Analysis

## 1. What is Greg?

Greg is a plant-care app and community focused on plant identification, personalized watering, reminders, plant care, and social/community support.

Greg's current App Store listing describes it as a "zero-guesswork plant care app & community."

Its core promise is to identify a user's plant, determine how much water it needs, and remind the user when it is time to water.

Source: Greg App Store listing — https://apps.apple.com/in/app/greg-plant-identifier-care/id1512912236

## 2. Plant Identification — PlantVision

Greg has a plant-identification feature called PlantVision.

The App Store listing says PlantVision can:

- Identify the plant species
- Provide plant information
- Measure/estimate pot size
- Estimate distance to the nearest window
- Estimate window orientation

Conceptually:

```text
Plant photo
    ↓
PlantVision
    ↓
Species identification
    +
Pot size
    +
Distance to window
    +
Window orientation
    ↓
Plant profile
```

## 3. Personalized Watering

Watering is the central feature of Greg.

Greg says it creates a custom watering plan based on:

- Plant species
- Plant size
- Details of the user's home environment

The App Store listing also describes "Precision Watering" as telling users how much H2O each plant needs and providing smart watering reminders.

## 4. User Feedback and Learning

Greg's current App Store reviews and developer responses indicate that the watering schedule adapts based on user inputs over time.

Greg's developer response explains that the schedule is calibrated for each plant using species, pot, light, and local weather, and learns from inputs such as snoozing a watering reminder or marking a plant as watered early.

```text
Recommendation
      ↓
User observes actual condition
      ↓
User action / feedback
      ↓
Greg updates the plant's state/model
      ↓
Future recommendation changes
```

## 5. Light and Environment

Greg includes an Indoor Light Gauge in its paid "Super Greg" feature set.

The App Store listing describes it as helping users ensure that their plants are getting enough sunshine.

PlantVision can estimate distance to the nearest window and window orientation.

## 6. Seasonal Updates

Greg's paid feature set includes "Seasonal Updates." The App Store describes this as adapting plant-care recommendations as the weather in the user's area changes.

```text
Same plant
    +
Same home
    +
Different season/weather
    ↓
Different care recommendation
```

## 7. Fertilizing and Other Care

Greg is not limited to watering. Its App Store listing and user reviews mention watering, fertilizing, plant progress/life tracking, and other plant-care activities.

## 8. Community

Community is a major component of Greg. Users can ask plant-care questions, get help from experienced growers, participate in communities, share plant-related content, and interact with other plant owners.

## 9. Does Greg Use Physical Sensor Data?

### Short answer: No evidence of a physical IoT sensor in the current product.

I could not find evidence in Greg's current public App Store listing or publicly surfaced product information that Greg uses dedicated physical:

- Soil-moisture sensors
- Soil-light sensors
- IoT plant sensors
- Bluetooth soil sensors
- Zigbee sensors
- A central hardware hub

Instead, Greg's current product description emphasizes software-based personalization using plant species, plant size, home environment, light/location information, pot information, user inputs, and weather/seasonal information.

PlantVision can infer/estimate physical characteristics such as pot size, window distance, and window orientation from the user's phone/camera.

Therefore the current architecture appears to be:

```text
Plant
  ↓
Plant profile
  ↓
User-provided / inferred environment
  ↓
Weather + seasonal information
  ↓
User actions / feedback
  ↓
Greg's software model
  ↓
Recommendation
```

NOT:

```text
Plant
  ↓
Physical sensor
  ↓
Continuous measurements
  ↓
Hub
  ↓
Cloud
  ↓
Recommendation
```

## 10. Greg's Intelligence Model

A simplified representation of the publicly described system is:

```text
                    PLANT
                      |
              +-------+-------+
              |               |
           Species          Size
                              |
                     +--------+--------+
                     |        |        |
                   Pot      Light    Location
                              |
                              ↓
                     HOME ENVIRONMENT
                              |
                    +---------+---------+
                    |                   |
                  Weather            Season
                    |                   |
                    +---------+---------+
                              |
                              ↓
                     USER FEEDBACK
                              |
                    +---------+---------+
                    |                   |
                 Watered            Snoozed
                    |                   |
                    +---------+---------+
                              |
                              ↓
                    GREG SOFTWARE MODEL
                              |
                              ↓
                 PERSONALIZED CARE PLAN
                              |
                    +---------+---------+
                    |                   |
                 When to water       How much
```

The exact internal algorithm is proprietary. This diagram represents the inputs and outputs Greg publicly describes, not its exact implementation.

## 11. The Key Difference Between Greg and Our Idea

Greg already does a large part of what we initially thought would differentiate us:

> "Use plant species + environment + historical actions to personalize care."

Therefore:

> "Personalized plant care" is NOT a sufficient differentiator.

The potential difference is the source and quality of the environmental data.

### Greg

```text
Plant
  ↓
Plant profile
  ↓
User enters / app estimates environment
  ↓
Weather + season
  ↓
User records watering
  ↓
Software model
  ↓
Recommendation
```

### Our potential system

```text
Plant
  ↓
Plant profile
  ↓
Continuous physical measurements
  ├── Soil moisture
  ├── Light
  ├── Temperature
  └── Humidity (later)
  ↓
Historical sensor data
  ↓
Plant-care engine
  ↓
Recommendation
```

## 12. The Most Interesting Opportunity

Greg predicts plant water requirements. Our system could potentially measure the plant's actual current environment.

### Greg

```text
Plant:
Monstera

Environment:
Low light
Large pot
Recent watering
Weather conditions

        ↓

Model prediction:
"Probably don't water yet."
```

### Our system

```text
Plant:
Monstera

Actual moisture:
48%

Actual light:
1,200 lux

Moisture history:
Drying 35% slower than usual

        ↓
Model + actual measurements
        ↓
"Don't water yet."
"Soil is drying slower than this plant's normal pattern."
"Light is also lower than usual."
```

This is a potential advantage, not a proven advantage. We need to test whether actual sensor measurements materially improve recommendations.

## 13. Manual State vs Automatic State

Greg depends on user interaction to maintain an accurate state.

```text
User waters plant
      ↓
User opens app
      ↓
User records watering
      ↓
Greg updates plant state
```

Our system could eventually detect changes automatically:

```text
User waters plant
      ↓
Moisture sensor detects sudden moisture increase
      ↓
System recognizes watering event
      ↓
Plant state automatically updated
```

This could eliminate a common source of error: the software thinks the plant has not been watered because the user forgot to record it.

Again, this is a product hypothesis that needs validation.

## 14. Example of the Potential Advantage

```text
Monstera

Normal behavior:
After watering → moisture falls steadily over ~5 days

Current behavior:
After watering → moisture remains high for 8 days

Additional observation:
Light is 30% lower than normal
```

A sensor-based system could potentially say:

```text
⚠️ Your Monstera is behaving differently.

Soil is staying wet ~60% longer than its normal pattern.

Light exposure is also lower than normal.

Likely explanation:
Reduced light may be slowing water consumption.

Recommendation:
Do not water yet.

Consider moving the plant closer to the window.
```

This is more than a reminder. It is an environmental diagnosis based on actual measurements.

## 15. Competitive Reality

Greg already has:

- Plant-specific care
- Personalized watering
- Light information
- Seasonal adaptation
- User feedback
- Historical watering behavior
- Plant identification
- Pot-size estimation
- Window-distance estimation
- Window-orientation estimation
- Plant community

Therefore we should NOT build the product around:

> "AI plant care."

Greg already does that.

We should NOT build the product around:

> "Personalized plant care."

Greg already does that too.

The potential differentiation is:

> **Continuous, objective, physical measurements of the plant's actual environment.**

## 16. Greg vs Planta vs FYTA vs Our Idea

```text
                     SOFTWARE INTELLIGENCE
                            ↑
                            |
                      Greg ●
                            |
                     Planta ●
                            |
                            |
                            |             FYTA ●
                            |
                            |
                      OUR TARGET ●
                            |
                            +------------------------→
                               PHYSICAL SENSOR DATA
```

### Sustee

```text
Cheap
Simple
Moisture-focused
Physical measurement
Very little intelligence
```

### Planta

```text
Strong software
Plant-specific care
Weather
User feedback
No continuous soil sensor
```

### Greg

```text
Strong watering intelligence
Plant-specific care
Light/location information
Seasonal adaptation
User feedback
Plant community
No evidence of core physical soil sensors
```

### FYTA

```text
Physical sensors
Central hub
Soil moisture
Light
EC
Temperature
Plant-specific software
Historical sensor data
Personalized recommendations
```

### Our potential product

```text
Low-cost sensor network
        +
Central hub
        +
Continuous moisture/light measurements
        +
Plant-specific intelligence
        +
Historical individual-plant behavior
        +
Actionable explanations
```

## 17. The Most Important Product Question

> **Can continuous physical sensor data produce meaningfully better plant-care decisions than Greg's software-only model?**

If the answer is NO:

```text
Greg's software
+
Cheap sensor
=
Unnecessary complexity
```

If the answer is YES:

```text
Greg-like intelligence
+
Actual soil/light measurements
+
Automatic state tracking
=
Potentially better product
```

## 18. What We Should Test

We should specifically test whether sensors improve these decisions.

### A. Watering

Can we know the actual soil state rather than predict it?

### B. Light

Can we continuously measure actual light rather than rely on user-entered location/window information?

### C. Watering events

Can we automatically detect when the user watered the plant?

### D. Drying curve

Can we learn how quickly each individual plant/pot/soil combination dries?

### E. Anomalies

Can we detect:

- Soil staying wet too long
- Soil drying unusually quickly
- Sudden changes in light
- Persistent low-light conditions

### F. Explanations

Can we say:

> "Don't water because the soil is still wet."

instead of:

> "Don't water because our model predicts you probably shouldn't."

## 19. Core Product Thesis After Studying Greg

The product should NOT be:

> "An AI plant-care app."

Greg already does that.

The product should NOT be:

> "A personalized plant-care app."

Greg and Planta already do that.

The stronger thesis is:

> **"A low-cost sensor network that continuously observes the real physical conditions of every plant and feeds those measurements into a plant-specific intelligence engine."**

The hardware is still not the product.

The sensors are the data-collection layer.

The product is the combination of:

```text
REAL-WORLD SENSOR DATA
          +
PLANT KNOWLEDGE
          +
INDIVIDUAL PLANT HISTORY
          +
PLANT-CARE INTELLIGENCE
          ↓
BETTER PLANT DECISIONS
```

## 20. Important Validation Rule

Do not build hardware just because we can.

For every sensor, ask:

> "What better decision can this measurement help us make?"

If continuous moisture + light measurements cannot provide meaningful advantages over Greg's software predictions, the IoT complexity is not justified.

The goal is not:

> "Build a smarter watering reminder."

The goal is:

> **"Measure what is actually happening to the plant, understand what it means for that particular plant, and tell the user what action to take."**

## Sources

- Greg App Store listing: https://apps.apple.com/in/app/greg-plant-identifier-care/id1512912236
- Greg official website: https://greg.app/
- Greg official support: https://greg.app/support/
- Greg official story: https://greg.app/story/
