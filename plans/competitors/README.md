# Competitor Analysis — Smart Garden

Consolidated from the original research notes (originals remain in git history at commit c346c6f). Research date: 19 Oct 2026. Company claims are unverified unless stated.

## 1. Bottom line

- **"Personalized / AI plant care" is not a differentiator.** Greg, Planta, Folly and Plant Parent all do it in software.
- **"We have sensors" is not a differentiator either.** FYTA, Botanical Legacy and LetPlant already combine sensors with plant-specific software.
- **The "many sensors → one hub → cloud → app" architecture is also taken.** FYTA already sells it (Grow Set €199.99 for 3 plants + room + hub; sensors from €29.99; hub €39.99), plus a B2B dashboard, API and integrations. See [profiles/fyta.md](profiles/fyta.md).
- What is left to test:
  1. **Cost / India-first economics**: ₹250–₹400 per plant node and about ₹500 hub, versus FYTA's roughly 8–10× higher EU prices (needs a BOM to back it).
  2. **Hub + plug-and-play**: no Home Assistant setup, one hub for 10–20 plants.
  3. **Decision quality**: turn telemetry into "3 plants need attention today", with explanations and anomaly detection, not dashboards.
  4. **Substrate-tuned sensors**: calibrated for common indoor mixes rather than arbitrary third-party sensors.
- **Central unproven hypothesis:** continuous sensor data produces materially better care decisions than software-only prediction. If it doesn't, don't build the IoT layer. See [validation-plan.md](validation-plan.md).

## 2. Landscape

| Product | Category | Own physical sensor | Continuous monitoring | Individual-plant model | Profile |
|---|---|---|---|---|---|
| Planta | Software-first | No | No | Yes (feedback-learned) | [planta](profiles/planta.md) |
| Greg | Software-first | No | No | Yes (feedback-learned) | [greg](profiles/greg.md) |
| Folly | Software-first | No evidence | No | Adaptive care | [folly](profiles/folly.md) |
| Plant Parent | Software-first | No (phone light meter) | No | Care history only | [plant-parent](profiles/plant-parent.md) |
| Botanical Legacy | Software + 3rd-party sensors | No (via Home Assistant) | Yes (~10 min poll) | Yes ("Digital Shadow") | [botanical-legacy](profiles/botanical-legacy.md) |
| LetPlant | AI app + own hardware | Yes (SDESA, rolling out 2026) | Yes | Yes | [letplant](profiles/letplant.md) |
| FYTA | Sensor + hub + app + B2B platform | Yes (Mini, Beam G2, Sphere, Terra; BLE + Wi-Fi hub) | Yes | Yes | [fyta](profiles/fyta.md) |
| Sustee | Cheap moisture sensor | Yes | — | Very little | benchmark only, no dedicated file yet |

Market progression: basic reminders → personalized software care → context-aware care → physical sensing → sensor + per-plant model → predictive / anomaly-aware management. The open question is where the next opportunity sits.

### Feature matrix (detail)

| Capability | Botanical Legacy | Folly | Plant Parent | LetPlant | Greg | Planta |
|---|---|---|---|---|---|---|
| Plant ID | Yes | Yes | Yes | Yes | Yes (PlantVision) | Yes |
| Photo diagnosis | Yes | Yes (Plant Doctor) | Yes | Yes | Not covered | Yes (Dr. Planta) |
| Personalized watering | Yes | Yes (Smart Care) | Yes | Yes | Yes | Yes (30+ parameters) |
| Weather / season | Yes | Yes | Yes | Yes | Yes (paid) | Yes |
| Light guidance | Yes | Yes | Phone light meter | Yes | Indoor Light Gauge (paid) | Light Meter (paid) |
| Soil-moisture sensor | **Yes** (3rd-party) | No evidence | No evidence | **Yes** | No | No |
| Light sensor | **Yes** | No evidence | Phone only | **Yes** | No | No |
| Temp / humidity sensor | **Yes** | No evidence | No evidence | **Yes** | No | No |
| Conductivity (EC) | **Yes** | No evidence | No evidence | Not established | No | No |
| Auto watering-event detection | **Yes** | Not established | Not established | Not established | No (manual log) | No (manual log) |
| Room-level sensing | **Yes** | Context only | No | Not established | No | No |
| Sensor-failure fallback to model | **Yes** | N/A | N/A | Not established | N/A | N/A |
| Home Assistant integration | **Yes** | No | No | Not established | No | No |
| Community | — | — | — | — | **Strong** | — |

## 3. Priority

- **Tier 1, study deeply (closest to our idea):** Botanical Legacy, LetPlant, FYTA.
- **Tier 2, study for software intelligence and UX:** Greg, Planta, Folly, Plant Parent.

## 4. Ideas worth borrowing

| Idea | From |
|---|---|
| Per-plant "Digital Shadow" model; a real reading overrides the model estimate | Botanical Legacy |
| Auto-detect watering from a moisture jump (e.g. 25% → 58%) | Botanical Legacy |
| Room-level temp/humidity sensor shared by many plants (cuts cost) | Botanical Legacy |
| Sensor failure lowers confidence instead of breaking the app | Botanical Legacy |
| What-if simulation (water / move / feed) | Botanical Legacy |
| Snooze / "watered early" feedback loop | Greg, Planta |
| Pot size, window distance and orientation from the camera | Greg (PlantVision) |
| Multiple growing sites with per-site weather alerts | Folly |
| Care workflow breadth, light meter, journaling | Plant Parent |
| Proactive "before visible stress" alerts | LetPlant |

## 5. Claims to avoid until proven

- "More personalized than Greg."
- "More accurate sensors than FYTA."
- "Cheaper than every competitor."
- "Continuous sensor data always gives better recommendations."

## 6. Open gaps in this research

- **Sustee has no dedicated file.** FYTA now has a profile, but it lacks sensor accuracy, calibration and sampling details. A physical-sensing deep dive on FYTA, Botanical Legacy sensors and LetPlant/SDESA is the next task. See the comparison dimensions in [validation-plan.md](validation-plan.md#next-research-task).
- **Pricing is mostly missing.** Botanical Legacy and PlantIn are now covered in [the deep dive](botanical-legacy-sensor-research.md#5-pricing-and-plans). The rest are not.
- **New competitor spotted: PlantIn** (software-only, mentioned in Botanical Legacy's comparison). It has no profile yet.
- **Key finding from the deep dive:** cheap capacitive sensors need per-soil calibration. An external 16-bit ADC cut RMSE to about 0.6–2.6% VWC in one lab study. Botanical Legacy's "over 5% unit-to-unit" claim is overstated: the cited paper attributes that gap mostly to uneven wetting. Unit-to-unit variance in potting mix is untested by anyone, so we should test it.
- **LetPlant SDESA** price, availability and shipping are unverified.
- **India availability** needs the clean-up noted in [india-availability.md](india-availability.md).

## Files

| File | Contents |
|---|---|
| [profiles/](profiles/) | One file per competitor |
| [botanical-legacy-sensor-research.md](botanical-legacy-sensor-research.md) | Deep dive: sensor quality findings, override rules, pricing, competitor claims |
| [india-availability.md](india-availability.md) | App-store and hardware availability in India |
| [validation-plan.md](validation-plan.md) | Differentiation, hardware architecture, sensor and software-vs-sensor experiments |
| [sources.md](sources.md) | All source links |
