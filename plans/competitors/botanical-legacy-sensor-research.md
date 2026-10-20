# Botanical Legacy — Sensor & Model Research Notes

Source: the Botanical Legacy blog post on plant moisture sensors and nine linked pages (listed at the bottom), fetched 20 Oct 2026. Overview in [profiles/botanical-legacy.md](profiles/botanical-legacy.md).

> **Bias warning:** every page is written by Botanical Legacy, so product claims and competitor comparisons are self-reported. None of the pages gives sensor accuracy data, calibration procedures, or model accuracy. Treat numbers marked "illustrative" as examples, not measurements.

## 1. Sensor landscape (their taxonomy)

| Type | Examples | Pros | Cons | Fits |
|---|---|---|---|---|
| Bluetooth probe | Xiaomi Mi Flora, HHCC Plant Genius / "Flower Care", Xiaomi 4-in-1 clones (under €20) | Cheap, coin-cell, "consistent enough to track a trend" | Needs a phone or relay nearby to reach the cloud | Small collections, one room |
| Wi-Fi probe (often ESP32) | DIY or pre-assembled capacitive probe | No hub | Shorter battery, one device per pot, you maintain firmware | 5+ plants, outdoor pots, builders |
| Hub-connected (Zigbee / Thread) | Aqara, Sonoff | Mesh reaches further, low power | Needs a smart-home hub (extra cost) | Larger collections, homes with a hub |

- **Mi Flora measures four channels in one stick:** soil moisture, light, temperature, EC (conductivity, a rough nutrient proxy). It uses Bluetooth, and Home Assistant needs firmware 3.2.1 or later via the Xiaomi BLE integration.
- **Capacitive vs resistive:** prefer capacitive (reads soil's effect on an electric field). Resistive probes have two bare metal prongs that corrode.
- **A bare capacitive board costs about USD 2**, barely more than a resistive one (per a 2024 peer-reviewed comparison, Nandi & Shrestha). Price is not a reliable quality guide.
- **Protocols:** Bluetooth (Mi Flora), Zigbee (common for soil probes), Matter/Thread (emerging, mostly at hub level; not yet relevant for soil probes). Home Assistant unifies them. Local API or MQTT is preferred over cloud-only.
- **Most branded plant apps can't read a Mi Flora** without Home Assistant or MQTT.

## 2. Sensor quality: what is known and unknown

- **Repeatability within one unit is good.** Repeated readings barely move. Verified in Placidi et al. 2020 (standard deviation always under 3.3 mV); see section 8.
- **Unit-to-unit variance:** Botanical Legacy says "two units of the same model read more than 5 percent apart". **After reading the source (Placidi 2020), this is overstated.** The 5% gap was between two sensors in the same lab sand sample, and the authors blame mainly non-uniform water distribution in the sample, not the sensors. The sensors' own electrical spread was small (1% frequency, 2.2% duty cycle), with one outlier of 14 units. Still, no one has measured unit-to-unit variance in real potting mix, so we should test it ourselves.
- **Long-term in-pot durability is an open question**, and neither paper tests it (corrosion, drift). We need to test it ourselves.
- **Absolute accuracy matters less than consistency:** the app tracks the drying trend, not the exact value. Fixed thresholds ("alert below 30%") are "species averages in disguise".
- **The sensor can't tell why a reading changed** (watering, rain, or a probe pushed deeper).
- **Placement is a source of error.** Probe about a third of the way down the pot, away from the wall and not just under the surface. One probe measures one spot, and software can't correct placement errors.
- **Spot-check readings against a finger test** occasionally.
- **Which channels are worth it:** moisture alone may be enough for one plant. Light and temperature drive drying rate, and EC signals when to feed.
- **Poor candidates for soil probes:** bromeliads, mounted staghorn ferns, air plants, water-grown lucky bamboo. Use room sensors for these.
- **Species differences change what a reading means.** Spider plants store water in tuberous roots, so 20% soil is less urgent than for a Boston fern.

## 3. How Botanical Legacy uses sensor data

### Model ("Digital Shadow")

- A per-plant model keeps a running estimate of remaining pot moisture and its drying rate. It draws down daily and **recalculates nightly**.
- **Inputs:**
  - time since watering (watering resets the estimate)
  - temperature, humidity, light
  - local weather (paid)
  - room-sensor readings
  - photo-derived data (height, leaf count/canopy, visible stress)
  - species, pot, soil
  - recovery state (e.g. root rot lengthens the interval)
- **30-day window:** previous 30 days of temperature, humidity, light and conductivity feed the nightly recalculation.
- **Learning:** each watering cycle adjusts a per-specimen correction factor. A postponed watering means the plant dries slower than predicted. The vendor says it takes "a couple of weeks" to converge.
- **Reminder** fires when the estimate crosses a per-plant threshold. The threshold value and how it is set are not disclosed.
- **No disclosed formulas, learning rates, confidence scores or accuracy figures.** The weight a reading gets against the estimate is not stated. They only say it "anchors" the model.
- **Their pitch:** a physics-style depletion model, not a calendar. Care forecast up to 7 days.

### Sensor-override rules (from the Home Assistant doc)

| Rule | Detail |
|---|---|
| Reading vs model | Fresh moisture reading **wins** (e.g. model 30%, probe 60% → 60%) |
| Deferral | A reading above the plant's "still wet" threshold pushes the next watering out (e.g. holds a day-7 watering when the probe reads 52%) |
| Conservatism | One odd reading shouldn't swing the schedule. Falls back to the model when readings are quiet or implausible |
| Room readings | Used at each recalculation only if under 4 hours old. Otherwise falls back to weather or standard indoor estimates |
| Watering alert | Moisture falls from above 20% to 20% or below → immediate alert |
| Auto-log watering | Rise of 25 percentage points or more **and** lands at 50% or wetter, and no watering logged in the last 6 hours |
| Offline | Silent 6 h → in-app/push alert. Silent 48 h → one email. Room sensors don't raise offline alerts. Falls back to the model, resumes smoothly |
| Drift flag | The daily "Sanctuary Brief" flags large model-vs-reading gaps (e.g. 20% vs 35% forecast) as possible calibration drift |
| EC | Banded for nutrition, trended, alerts on salt build-up |

### Pipeline limits worth noting

- Polled every **10 minutes**, history stored **hourly for 30 days**. Not a real-time feed.
- Only five Home Assistant device classes are accepted: temperature, humidity, moisture, illuminance, conductivity. Units are converted, and unrecognised units are skipped.
- **Binding:** moisture, light and EC bind to one plant. Humidity binds to a room. Temperature binds to either. A room reading applies to every plant in the room.
- **Setup friction (our opening):**
  - Home Assistant must be reachable from the internet over `https://`. Local addresses, Tailscale and ZeroTier don't work. You need Home Assistant Cloud, a reverse proxy, or a Cloudflare Tunnel.
  - You create a non-admin user and a long-lived token. They say setup takes about 10 minutes, and their own firmware for direct probes is "not ready for a wider audience".
  - Native standalone-probe integrations are only on their roadmap, and the status is dated June 2026.

## 4. Physics worth encoding in our model

From their care and heat-stress pages (general horticulture, not their model output):

- **Temperature strongly drives drying.** A Monstera in a 25°C room may drain its pot in about 5 days, and at 14°C may go about 3 weeks. Their own illustrative example moves the nudge from about day 9 (baseline) to about day 6 (28°C, dry air) or day 14 (19°C, cloudy).
- Heated homes in winter can drop below 30% RH, which speeds drying.
- **Pot and substrate:** terracotta dries faster than plastic or glazed ceramic. Chunky aroid mixes drain fast and peat-heavy mixes hold water. Smaller pots dry faster.
- **Heat stress:** growth slows above about 32°C and damage is likely above about 40°C. Succulents tolerate more, and calatheas and ferns less. Direct light adds radiant heat beyond air temperature. Dry soil reduces cooling because stomata close.
- **Watering behaviour by group:**
  - Tropicals: about 1–2×/week.
  - Ferns and calatheas: every 3–5 days in dry air.
  - Succulents and cacti: 2–3 weeks in summer.
  - Drought-tolerant plants (jade, aloe, snake plant, ZZ) tolerate lateness better than early watering.
- **Manual methods to benchmark against:**
  - Finger test, two knuckles deep (they call it "the single most reliable cheap test").
  - Lift test (weigh just after watering vs about 3 days later).
  - Surface dryness is unreliable because the top 1 cm dries first.
- **Overwatering recovery:** let the top third dry. If no improvement in 48 h, check roots.

**Implication for us:** room temperature and humidity are cheap shared signals with strong effects on drying, which supports the room-level node idea. Light and EC are optional per-plant channels.

## 5. Pricing and plans

Their pages disagree on the Cultivator price, so confirm before using it as a benchmark.

| Plan | Price | Specimens | Sensor connections | Photo reads/month |
|---|---|---|---|---|
| Observer | Free | 5 | None | 3 |
| Cultivator | €6.99/month (pricing page) or €49.99/year (PlantIn comparison page) | 50 | up to 50 | 120 |
| Conservancy | €14.99/month or €116.99/year | 150 | up to 150 | 360 |

- Sensor telemetry is **paid-only** (Cultivator or higher). The free plan has no sensors.
- Annual billing saves 35–40%. Prices are EUR plus VAT.
- 14-day Cultivator trial, 7-day Conservancy trial.
- Full data export on every plan. Household sharing and voice assistants (Alexa, Google) are on the roadmap.
- Cancelling pauses polling but keeps bindings.
- **They sell no hardware.** Their revenue is subscription only, so the sensor cost sits on the user.

## 6. Their claims about competitors (unverified)

| Competitor | Claim | Our note |
|---|---|---|
| Greg | Schedule is "city climate + species lookup", doesn't tune to individual plants, check-in photos don't feed the schedule, no sensors | Conflicts with Greg's developer response, which says the schedule learns from snoozes and early waterings ([profiles/greg.md](profiles/greg.md)). Don't rely on this |
| Greg | Strong community ("Plant Friends"); tens of thousands of species | Consistent with our notes |
| PlantIn | Calendar + notifications, no sensors, about $520/yr (website) or $29.99/yr (App Store); 14,000+ species | New competitor, software-only. The pricing spread is useful context for app-subscription willingness to pay |
| Planta | 25,000+ species, great design, "reminder-class at heart" | Their characterisation. Planta documents a feedback loop |
| PictureThis, Blossom | Best ID camera / quick disease checks | Not sensor competitors |

They also concede that others have larger species catalogues, cheaper weekly entry, and stronger community.

## 7. What this means for our plan

1. **Unit variance is unmeasured, not proven bad.** The "over 5 points" claim does not hold up (section 8). Still, the literature never tests unit-to-unit spread in potting mix, so buy 10+ units of the candidate capacitive sensor and test them. Per-plant baselines (learn dry/wet points per unit) are likely more practical than factory calibration. The papers also show cheap sensors improve a lot with a clean 16-bit ADC.
2. **Copy the override rules:** reading beats model, conservative deferral, a staleness window for room data, offline fallback, and a drift flag when model and sensor disagree persistently. These are concrete defaults.
3. **Auto-watering detection has a ready heuristic** to start with: rise of 25 points or more, landing at 50% or wetter, with a 6 h debounce. It will misfire on non-watering changes, so log false positives in testing.
4. **Our opening is their setup friction:** an internet-reachable HTTPS Home Assistant, tokens, entity mapping. A dedicated hub (sensor → hub → app) avoids all of it.
5. **Sample rate:** a 10-minute poll and hourly history is enough for their model. Battery nodes can sample less often.
6. **Business model to compare:** they charge a subscription and sell no hardware. Our hardware margin vs subscription mix is still undecided.
7. **Nobody publishes accuracy data, calibration or durability.** If we measure it, that data is itself an asset.

## 8. Verification of cited references (read 20 Oct 2026)

The pages cite about 50 sources but never tie a claim to a source ("facts were checked against these references"). I read the two sensor papers and 9 of the extension/RHS pages.

### Sensor papers

**Nandi & Shrestha 2024, *Sensors* 24(18):5886** (University of Idaho). https://pmc.ncbi.nlm.nih.gov/articles/PMC11435841/

- **Tested:** FC-28 resistive (about USD 1.50), generic capacitive v1.2 (about USD 2), Vegetronix VH400 (about USD 42), Decagon 5TM (about USD 350), and a 16-bit ADS1115 ADC (about USD 5) fitted to the cheap sensors.
- **Method:** one silt loam sieved into 3 "textures", 0–50% VWC in 10% steps, 3 replications, third-order polynomial calibration, 20 °C only.
- **Result (RMSE, % VWC):**

| Soil | Capacitive | Capacitive + ADC | Resistive | Resistive + ADC | VH400 | 5TM |
|---|---|---|---|---|---|---|
| Coarse | 1.88 | 1.27 | 3.83 | 1.55 | 0.85 | 0.82 |
| Fine | 3.22 | 1.81 | 4.39 | 0.64 | 1.17 | 0.95 |
| Medium | 3.78 | 1.87 | 4.79 | 2.64 | 1.80 | 0.47 |

- **Maximum errors:** up to 4.7–8.5% for capacitive and 9.4–12.5% for resistive without the ADC.
- **Reading noise fell from about 27 mV to 0.95 mV** (capacitive) with the ADS1115.
- **Conclusion:** with the external ADC, cheap sensors were "comparable" to the expensive ones (RMSE differences not statistically significant).
- **Not tested:** unit-to-unit variability (one unit per type), temperature, salinity, durability, real potting mix, or a separate validation set.
- **Inconsistency:** the conclusions quote slightly different numbers from Table 1.
- **Confirmed:** the "about USD 2" capacitive board and the claim that it costs barely more than resistive.

**Placidi et al. 2020, *Sensors* 20(12):3585** (University of Perugia). https://pmc.ncbi.nlm.nih.gov/articles/PMC7348898/

- **Tested:** the DFRobot SEN0193 (capacitive v1.2) in oven-dried silica sand, 14 units (9 electrically characterised, 5 modified).
- **Findings:**
  - Output vs gravimetric water content is non-linear (correlation −0.945), so it needs a polynomial or exponential fit.
  - Repeatability from a single sensor is good (SD under 3.3 mV).
  - Compaction and porosity strongly change readings. The sensing volume is unknown.
  - Calibration is valid only for one soil type and density.
  - Temperature and salinity effects were not tested but are named as complications.
  - Design flaws: the 555 timer is driven near its frequency limit with a duty cycle of about 33%, and older v1.2 boards lack a ground path for one resistor.
- **Source of the ">5%" claim:** two sensors in one sample differed "even more than 5%", attributed mainly to non-uniform water distribution. The electrical spread across units was 1% in frequency and 2.2% in duty cycle, with one outlier (S1).
- **Note:** this is also the sensor most commonly used in DIY projects, and our candidate.

### Horticulture sources (what they actually say)

| Claim on BL pages | Source check | Result |
|---|---|---|
| Probe about a third of the way down the pot | RHS container page ("push your finger about a third of the way down the pot") | ✅ Confirmed, for a finger test. BL applies it to probe placement |
| Water when top 2–3 cm of compost is dry (containers) | RHS watering page | ✅ Different depth from the "third of the pot", so the depth rule is not one number |
| Finger test as the reference method | UMD and Illinois: 2 inches (about 5 cm). Clemson: first joint. UMD and Clemson also suggest weighing the pot | ✅ Multiple methods, and the depth varies by source |
| "Finger test is the single most reliable cheap test" | Not found in any source read | ⚠️ BL's own wording |
| Moisture meters are only a trend tool | Clemson: "accuracy varies widely", readings affected by fertilizer and soil type. Illinois: "may not always be reliable" | ✅ Supports validating sensors against a finger or weight test |
| Terracotta dries faster; larger pots hold water longer; peat-heavy mixes hold more | RHS, Clemson, Illinois | ✅ Confirmed |
| Factors in drying: species, pot size, light, temperature, humidity, mix, growth stage | Clemson, Illinois, UMD | ✅ Confirmed. Clemson adds dormancy and pot-to-plant size ratio |
| Peat-free compost needs slightly more frequent watering | RHS | New detail for substrate calibration |
| Tap water: stand 24 h to remove chlorine | UMD: 24 h; Clemson: 62–72 °F | ✅. BL adds that fluoride is not removed, and UMD says chlorine and fluoride "dissipate". These conflict |
| EC as a nutrition / salt-build-up signal | CSU: EC measures total soluble salts, but in a **soil extract (ECe)** with thresholds of 0–2 dS/m non-saline up to 16+ dS/m. UMD: leach at least one pot volume | ⚠️ Those thresholds are for soil extracts and field crops. A probe's in-pot EC reading isn't directly comparable, so BL's "banded" EC is unvalidated |
| Heat/humidity effects on drying | UMD and Clemson only name them as factors, with no numbers | ⚠️ "25 °C = 5 days, 14 °C = 3 weeks" is BL's own example, not sourced |

Other points from the sources:

- Clemson: the bottom two-thirds of the pot holds most roots, so probe depth matters.
- Clemson: wilting does not always mean thirsty, because root rot also wilts the plant.
- Illinois: plants don't suit a fixed calendar, so water on need.
- UMD: leaching every 4–6 months with at least twice the pot volume.

### Not verified

- **Egri et al. 2010** (*New Phytologist*): PubMed served a cookie wall. Title and findings unknown.
- About 35 other extension and RHS pages were not read (RHS plant-specific guides, USGS evapotranspiration, NC State, Iowa State, Penn State humidity, etc.). They are general horticulture, not sensor research. The most relevant to read next:
  - USGS evapotranspiration, for the drying physics
  - Penn State humidity
  - UF/IFAS light for houseplants
  - Oregon State heat stress
- The two roundups and `about` methodology page on Botanical Legacy are also still unread.

## Open questions

- What is the real Cultivator price (€6.99/month vs €49.99/year)?
- What is the actual threshold logic ("still wet" threshold per plant)?
- ~~Which peer-reviewed tests do they cite?~~ Answered in section 8. Placidi 2020 is the source of the ">5%" figure. Egri et al. 2010 could not be read (PubMed cookie wall).
- Their own firmware for direct probes: what does it support?

## Pages reviewed

- https://botanicallegacy.com/blog/plant-moisture-sensors-houseplants
- https://botanicallegacy.com/sensors
- https://botanicallegacy.com/blog/plant-care-app-with-sensor-support
- https://botanicallegacy.com/blog/your-plants-digital-shadow
- https://botanicallegacy.com/blog/connect-plant-sensor-home-assistant
- https://botanicallegacy.com/blog/greg-app-alternative
- https://botanicallegacy.com/blog/plantin-vs-botanical-legacy
- https://botanicallegacy.com/pricing
- https://botanicallegacy.com/blog/when-to-water-houseplants
- https://botanicallegacy.com/blog/plant-heat-stress-monitoring-app

Not yet read: `/blog/best-plant-care-app-2026` (roundup of popular apps), `/blog/predictive-plant-care`, `/about` (methodology), `/blog/indoor-plant-care-heat-wave`.
