# Plan: Build a multi-node moisture + light sensor prototype (Wi-Fi hub)

## Context

Build a working model before market work. The prototype answers the questions in `competitors/validation-plan.md`: are cheap capacitive sensors good enough, do several nodes reach one hub reliably, and does sensor data improve watering decisions? It is not a product. Optimise for learning speed, not the ₹250–₹400 BOM (Stage 6).

Decisions made:
- **Wi-Fi hub from the start, no USB to the laptop.** The hub is wall-powered and posts readings over home Wi-Fi.
- **Data goes to a small Python server on the laptop**, which appends to a CSV. The laptop must be on and on the same Wi-Fi during tests.
- **Arduino IDE** for compiling and flashing (no separate C++ install; the IDE bundles the compiler).

Departures from `plant_monitoring_plan.txt`:
- **ESP-NOW (ESP32 to ESP32) between nodes and hub, instead of 433 MHz or BLE.** Same chip and toolchain on both sides, built-in ACK and retry, no extra radio modules, nodes can deep-sleep. Revisit 433 MHz in Stage 6.
- **BH1750 digital light sensor instead of an LDR**, so light is real lux. Compare an LDR against it in Stage 6.
- **Prototype nodes are ESP32 dev boards (₹400–600 each)**, so the node cost target isn't met yet.

## Architecture

```text
Node (ESP32 + capacitive moisture + BH1750)
   wakes every 1-10 min, reads, deep-sleeps
        │ ESP-NOW (2.4 GHz)
        ▼
Hub (ESP32, wall-powered, joins home Wi-Fi)
        │ HTTP POST JSON  {node, seq, moisture_raw, lux, batt_mv}
        ▼
Laptop: tools/server.py  ->  readings.csv
```

**Channel rule:** ESP-NOW only works if the node and the hub are on the same radio channel, and the hub's channel is whatever the router uses. Fix the router's 2.4 GHz channel (e.g. channel 6) in its admin page and set `ESPNOW_CHANNEL` to match in the node config. The hub prints its actual channel and warns if it differs.

## End-to-end flow

### 1. One-time setup (in order)

```text
YOUR MAC                                                 BOARDS / NETWORK
────────                                                 ────────────────
[1] Install Arduino IDE 2.x
        │
[2] Boards Manager: install "esp32 by Espressif" v2.0.17
    Library Manager: install "BH1750"
        │
[3] Router admin page ──────────────────────────────►  Fix 2.4 GHz Wi-Fi channel (e.g. 6)
        │                                              Note your Mac's IP (e.g. 192.168.1.20)
        │
[4] Edit hub/secrets.h: Wi-Fi name, password, Mac IP
        │
[5] Plug HUB ESP32 into Mac by USB ──────────────────►  Upload hub.ino
        │                                               Serial Monitor prints:
        │                                               "hub MAC: AA:BB:CC:DD:EE:FF, channel 6"
        │
[6] Copy hub MAC and channel into node/config.h
        │
[7] Plug NODE ESP32 into Mac ────────────────────────►  Set NODE_ID = 1, upload node.ino
    (repeat for node 2, 3, changing NODE_ID)            Wire moisture sensor + BH1750 to the node
        │
[8] Move HUB to a wall charger (no more USB) ────────►  Hub joins Wi-Fi, waits for nodes
        │
[9] Terminal: python3 hardware/tools/server.py   ◄───   Start the receiver on your Mac
```

### 2. What happens while running

```text
 NODE (battery, in the pot)                    HUB (wall power)                     MAC
 ──────────────────────────                    ────────────────                     ───
 deep sleep 1–10 min
      │ timer wakes
      ▼
 power moisture sensor on, wait 100 ms
 read ADC × 32, average  ─┐
 read BH1750 (lux)        ├─► build packet
 power sensor off         ─┘   {node_id, seq, moisture_raw, lux, batt_mv}
      │
      │  ESP-NOW (2.4 GHz radio, same channel as router)
      ├─────────────────────────────────────────►  receive callback
      │                                             put packet in queue
      │◄─────────────────────────────────────────  ACK sent automatically
      │  (no ACK: retry up to 3 times)                  │
      ▼                                                 ▼
 deep sleep again                                  loop(): take packet from queue
                                                        │
                                                        │  HTTP POST JSON
                                                        │  http://<mac-ip>:8000/reading
                                                        ├──────────────────────────►  server.py
                                                        │                                │ add timestamp
                                                        │                                ▼
                                                        │                            readings.csv
                                                        │  (Wi-Fi down? keep up to
                                                        │   ~100 packets in RAM, retry)
```

### 3. After data is collected

```text
readings.csv  +  daily pot weight (kitchen scale)  +  watering log
        │
        ▼
Calibration: raw moisture vs actual water content, per substrate
        │
        ▼
Compare: software-only model  vs  sensor-assisted model
        │
        ▼
Go / no-go (decision metric agreed before the 3–4 week test)
```

Failure points to watch:
- **Step 3:** if the router changes channel on its own, nodes stop reaching the hub and nothing shows up in the CSV.
- **Step 8:** the hub only needs USB for the first flash. After that it just needs power.
- **Step 9:** if the Mac sleeps or changes IP, the hub keeps buffering readings in RAM but loses them if it runs out of room or reboots. Give the Mac a fixed IP in the router.

## Toolchain: Arduino IDE

One-time setup:
1. Install Arduino IDE 2.x.
2. Boards Manager: install **"esp32" by Espressif Systems, version 2.0.17 exactly** (3.x changes the ESP-NOW function signatures and breaks this code). Board: "ESP32 Dev Module".
3. Library Manager: install **"BH1750" by Christopher Laws**.
4. If the board isn't recognised, install the CP210x or CH340 USB driver (check the chip next to the USB port).

## Scaffolding already in the repo (untested, not compiled)

Under `hardware/`, written for PlatformIO. Needs converting for Arduino IDE:
- `node/src/config.h` + `node/src/main.cpp`: read, send with 3 retries, deep sleep. **Convert to `node/node.ino`** (+ `config.h` in the same folder). `NODE_ID` becomes a plain line in `config.h` that you edit before flashing each node.
- `hub/src/main.cpp`: currently prints CSV over USB serial. **Convert to `hub/hub.ino` and change it** (below).
- `platformio.ini`: **delete.**
- `tools/logger.py`: serial logger. **Delete it**, replaced by `tools/server.py`.

## Changes to make

1. **`hub/hub.ino`:** join Wi-Fi in STA mode (credentials in a new `hub/secrets.h`, git-ignored, with `secrets.example.h` committed). Keep the queued ESP-NOW receive path and replace the Serial print with an HTTP POST (`HTTPClient`) to `http://<laptop-ip>:8000/reading`. If Wi-Fi or the POST fails, keep up to ~100 readings in RAM and retry. Print the hub MAC, channel and laptop IP on the serial monitor at boot (needed only for the first flash).
2. **`hardware/tools/server.py`:** ~30 lines, standard library only (`http.server`). Accepts the POST, adds a timestamp, appends to `readings.csv`, prints each row. Run with `python3 tools/server.py`.
3. **Remove `tools/logger.py` and `platformio.ini`.** Add `hub/secrets.example.h`, and a `.gitignore` entry for `secrets.h` and `readings.csv`.
4. **Write `hardware/README.md`** as a beginner build guide (parts, wiring, flashing, finding the laptop IP, setting the router channel, common mistakes).
5. **Compile-check from the terminal** with `arduino-cli` (installed via Homebrew, plus the same esp32 2.0.17 core) so errors are caught before flashing. You still flash with the Arduino IDE. This is a few hundred MB of downloads.
6. Add a pointer to the prototype in `competitors/validation-plan.md`.

## Stages

**Stage 1: One node + hub.**
- Minimum for Stage 1 alone is 2 ESP32 boards (1 hub + 1 node), but buy the full set below so Stages 2–3 don't wait on shipping. Three nodes is the useful number: two rooms, a couple of substrates, and visible packet loss.
- Shopping list (1 hub + 3 nodes, with spares):

  | Item | Qty | Why |
  |---|---|---|
  | ESP32 DevKit (30-pin, CP2102 or CH340 USB chip) | 5 | 1 hub, 3 nodes, 1 spare |
  | Capacitive soil sensor v1.2 (chip marked TLC555) | 6 | 3 for nodes, 3 more to compare units of one model |
  | BH1750 light module | 3 | 1 per node |
  | ADS1115 ADC module | 1 | Test whether an external ADC improves readings (Stage 2) |
  | Breadboards + jumper wires | 1 set | Wiring |
  | USB cables | 2–3 | Flashing and power |
  | Phone charger | 1 | Powers the hub |
  | Kitchen scale (0.1 g) | 1 | Pot weight as ground truth |

  Later (Stage 3 battery tests): 18650 cells + TP4056 charger boards. Expected spend is a few thousand rupees; prices not yet checked.
- Node wiring: sensor VCC to GPIO25, GND to GND, AOUT to GPIO34; BH1750 VCC to 3.3 V, GND to GND, SDA to GPIO21, SCL to GPIO22.
- Flash the hub once over USB (read its MAC from the Serial Monitor), put the MAC in `HUB_MAC`, flash node1. Plug the hub into a wall charger. Run `server.py` and watch rows arrive.

**Stage 2: Calibrate moisture.** Sensor in air, in water, and in dry / damp / saturated potting mix and cocopeat. Compare raw ESP32 ADC with the same sensor read through the ADS1115.

**Stage 3: Three nodes on real pots.** Range through walls, packet loss (gaps in `seq`), and battery life on 18650 + TP4056.

**Stage 4: The experiment.** 8–10 plants for 3–4 weeks. Weigh each pot daily and log waterings. Compare the software-only model with the sensor-assisted one. Define "better decision" up front.

**Stage 5: Phone access (only if Stage 4 says go).** Move the sink from the laptop to the cloud or MQTT/Home Assistant.

**Stage 6: Cost down.** ESP32-C3 SuperMini nodes, LDR vs BH1750, 433 MHz vs ESP-NOW, and a costed BOM against ₹250–₹400 per node and ~₹500 hub.

## Known gotchas
- Moisture-sensor chip should say **TLC555**. An NE555 clone won't run at 3.3 V.
- **ADC1 pins only** (GPIO32–39).
- **Seal the sensor's exposed electronics** and insert only to the marked line.
- Capacitive output is **higher when drier**. The ESP32 ADC is non-linear and noisy, so the node averages 32 samples and you test the ADS1115.
- Dev boards draw far more in sleep than a bare module, so battery numbers will be pessimistic.
- Calibrate per substrate and per pot.
- ESP-NOW and Wi-Fi share one radio, so a router that changes channels automatically will silently break the link. Fix the channel.
- Never commit `secrets.h` (Wi-Fi password).

## Verification
- **Compile:** `arduino-cli compile` succeeds for both `hub/` and `node/` (esp32:esp32:esp32, core 2.0.17).
- **Stage 1 pass:** `readings.csv` gets a new row from the node at the chosen interval with `seq` increasing by 1. Moisture changes when you wet the probe and lux changes when you cover the BH1750.
- **Hub resilience:** disconnect the laptop's Wi-Fi for 2 minutes, reconnect, and the buffered rows appear.
- **Stage 2 pass:** raw readings fall steadily from dry to wet and repeat within a small band across 10 readings.
- **Stage 3 pass:** over 48 hours, fewer than 5% of `seq` numbers are missing per node.
- **Stage 4 pass:** the decision metric you agree before starting.

## Open items
- OK to install `arduino-cli` with Homebrew to compile-check the firmware? (Skippable: you would then find compile errors yourself in the IDE.)
- Which plants and substrates for Stage 4?
