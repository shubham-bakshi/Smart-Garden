# Adjacent Players & Hardware References (India)

From a list of links pasted by the user (titles only, no URLs, likely from an AI chat). Each was searched for on 20 Oct 2026. Several pages could not be opened (403 or search snippets only), so details below are from search results and are unverified.

| Item from the list | Found? | Verdict |
|---|---|---|
| Research and Markets, India Indoor Plants Market to 2030 | No | The report wasn't found. See [india-indoor-plants-market.md](india-indoor-plants-market.md) for the three reports that were found |
| Ugaoo indoor plants catalog | Yes | Plant seller, not a competitor |
| Sonoff plant monitoring guide | No | No official guide found. Sonoff doesn't appear to sell a soil probe |
| IISc Embedded Lab indoor plant monitoring system | No | No IISc project found. Unrelated university papers turned up instead |
| YourStory, Agro2o | Yes | Hydroponic home garden device, Delhi |
| The Better India, BitMantis | Yes | GreenSAGE hydroponic kits, Bengaluru |
| YourStory, 4Climate | Partly | B2B farm automation, Gujarat |

## What was found

### Ugaoo (plant retailer)
- Pune-based online plant store founded in 2015. It sells plants, planters, seeds and care products through its own site and marketplaces such as Amazon and Flipkart.
- Per TechSci: 1M+ orders in three years and a plan to cultivate 500,000 plants a month.
- **Relevance:** a possible bundling or distribution channel, and a source of data on post-purchase plant death.

### Agro2o (Delhi, MeitY-backed)
- **Product:** a fully automated indoor hydroponics device. Pre-seeded pods plus a nutrient cartridge, a touchscreen, Wi-Fi and a phone app. It adjusts light, water and nutrients by growth stage and reminds you to refill the water tank. Up to 50 plant types.
- Founder: Yash Vyas, started 2018.
- **Funding (StartupWire, 9 Aug 2026):** led by Mumbai Angels Network, amount **not disclosed**. Proceeds go to hardware scaling, R&D, manufacturing in Delhi-NCR, B2B installations and better AI nutrient dosing.
- **Current pitch:** sensors for water pH, EC and humidity plus spectrum-tuned LEDs and automatic nutrient dosing. Modular vertical units for B2B (commercial kitchens, offices, hotels, supermarkets, cafeterias) and homes. B2B looks like the priority.
- Company-reported claims, unverified: about 90% less water, up to 3× faster growth, 5× yield per square foot. No revenue, unit sales, customer numbers or price were given.
- **Relevance:** it solves the care problem by controlling the environment of a closed hydroponic unit, not by monitoring ordinary potted plants. It's a different product from ours, but it shows Indians will pay for automated home growing. Its price wasn't found.
- Claims such as "plants grow about 5× faster" come from a press release and are unverified. The YourStory article returned 403, so I used search snippets.

### BitMantis (Bengaluru)
- IoT and analytics company whose **GreenSAGE** product has a Micro Edition (microgreens, two trays) and a Greens Edition (herbs and salad greens) with replaceable trays, for homes and commercial growers.
- Pricing and availability not found. Hydroponics, so likewise a different product.

### 4Climate (Vadodara, Gujarat)
- Controllers and nutrient-dosing units for greenhouses and vertical farms. Control logic runs on-site (edge) rather than in the cloud.
- Reported scale varies: 800,000 sq ft and 75+ installations (NITI portal) vs 500,000 sq ft across 15 projects (earlier interview). Founded 2023 per YourStory, 2020 per another database. Funding figures conflict ($58K, $190K, $800K).
- **Relevance:** B2B, commercial-farm focus. Not a consumer competitor, but it shows Indian demand for affordable automation. Verify before citing.

### Sonoff / Zigbee soil probes (hardware reference)
- **Sonoff** doesn't seem to make a soil moisture probe. It has Zigbee 3.0 temperature/humidity sensors (SNZB-02D), bridges (ZBBridge) and a Hydro ONE water valve.
- **ThirdReality Smart Soil Moisture Sensor Gen2:** USD 19.99 list (USD 14.99 sale), Zigbee 3.0, capacitive, one AA battery, needs a Zigbee hub (Home Assistant, Hubitat, SmartThings, Homey). No accuracy or battery life stated.
- **NEO NAS-STH02B2:** solar-powered probe for soil temperature, humidity and EC, IP66, works with Zigbee2MQTT.
- **Relevance:** a US$15–20 retail price for a Zigbee soil sensor is another benchmark for our ₹250–₹400 node (about US$3–5 at an assumed ₹85/US$, to be checked). It also shows Zigbee sensors still need a separate hub, which is the setup friction our all-in-one kit removes.

## Not found
- The IISc embedded-lab project. The search surfaced unrelated university papers (Politecnico di Torino impedance study, a Cartagena IoT architecture, an arXiv winter-survival project, a TU Sofia ESP32 prototype). Check IISc's ECE or CEDT sites directly if this reference matters.

## Takeaways
1. **No Indian consumer product monitors potted indoor plants with sensors.** The Indian players found are either plant sellers or hydroponic/farm automation. This is only the absence of results from a few searches, not proof.
2. **Hydroponic home kits are a different category** where the device controls the environment. Our product observes ordinary plants.
3. Retailers (Ugaoo, Nurserylive, Urvann, Kyari) are the more likely partners.
4. The user's list came from an AI response with no URLs, and two of seven entries could not be found, so don't cite any of it without checking.
