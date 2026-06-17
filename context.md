# RingForge — Context & Data Model

This document records **everything the configurator assumes**: every sensor, what it
does, what it unlocks, and how it affects the three live outputs (battery life, weight,
cost). It separates **real, verifiable data** from **modeled estimates** so nothing in
the UI is mistaken for a vendor datasheet.

See `research.md` for the full external technical audit (June 2026) that these values
were checked against. The corrections from that audit are folded in below.

Last updated: 2026-06-17.

---

## 1. What this tool is

A Miro-style whiteboard where you "build" a custom smart ring by toggling sensor cards
arranged around a 3D render of a real titanium ring. Selecting a card means "include
this in my ring." The right-hand readout updates live:

- **Battery life** (days per charge) — shown as a range
- **Weight** (grams) — shown as a range
- **Est. parts cost** (rough) — shown as a range
- **Unlocked metrics** — what the chosen sensors can measure

The whole point is to make the **engineering trade-offs** visible, especially the way
power-hungry features (above all, a continuously listening microphone) collapse battery
life.

---

## 2. The single most important assumption

A real smart ring runs its **entire system** — every sensor, the microcontroller, the
Bluetooth radio, the power-management IC — on roughly **1.5–3 mAh per day**.

Why: shipping rings carry **17–22.5 mAh** cells and last **5–14 days**. Example:
22.5 mAh ÷ 7 days ≈ 3.2 mAh/day; RingConn's 18-ish mAh ÷ 12 days ≈ 1.5 mAh/day.

This is the hard physical envelope. Any per-sensor number that, when summed, blows past
~3–4 mAh/day would imply a sub-2-day ring, which doesn't exist. So the model is built
around a **platform baseline** plus **small marginal sensor draws**, not large
per-sensor numbers.

---

## 3. How each output is computed

Let the selected set of components be S, the chosen battery cell B, and a fixed
**platform baseline** (always present: MCU + BLE radio + PMIC + firmware).

```
PLATFORM BASELINE   draw = 1.6 – 2.6 mAh/day   weight = 1.6 – 4.2 g   cost = $40 – $110
```
(Weight floor lowered from 2.3 g to 1.6 g so a minimal build can approach the ~2 g of the
Oura Ring 5; ECG-equipped builds reach ~5 g.)

**Battery life (range, days):**
```
daily_draw_low  = baseline_low  + Σ(sensor_draw_low  for sensor in S)
daily_draw_high = baseline_high + Σ(sensor_draw_high for sensor in S)

days_max = cell_mAh / daily_draw_low      (best case — sensors sip power)
days_min = cell_mAh / daily_draw_high     (worst case — sensors run hot)
```

**Weight (range, g):**
```
weight_low  = baseline_g_low  + cell_g + Σ(sensor_g)
weight_high = baseline_g_high + cell_g + Σ(sensor_g)
```

**Cost (range, $):**
```
cost_low  = baseline_cost_low  + cell_cost + Σ(sensor_cost_low)
cost_high = baseline_cost_high + cell_cost + Σ(sensor_cost_high)
```

The dominant term in every case is the **baseline**, not the individual sensors — which
is the honest reality. Sensors are tiny and cheap relative to the radio, battery, and
titanium shell.

---

## 4. Battery cells (REAL DATA)

These are the actual Samsung Galaxy Ring capacities, which scale with ring (finger)
size. Oura keeps its mAh figure proprietary but uses the same class of small lithium-
polymer cell.

| Option | Capacity | Finger sizes | Added weight | Added cost |
|--------|----------|--------------|--------------|------------|
| Small  | 17 mAh   | sizes 5–7    | 0.4 g        | $9         |
| Medium | 18.5 mAh | sizes 8–11   | 0.6 g        | $11        |
| Large  | 22.5 mAh | sizes 12–13  | 0.9 g        | $14        |

Real-world battery life for reference (2026 lineup):
- Oura Ring 5 (shipped Jun 4 2026): **6–9 days**, **from 2 g** (max 2.69 g), $399 base — current flagship
- Oura Ring 4 (still sold): **5–8 days**, charge 20–80 min, titanium, weight 3.3–5.2 g
- Galaxy Ring: **6–7 days** (Galaxy Ring 2 not yet out — expected early 2027)
- RingConn Gen 2: **10–12 days**; Gen 3: up to **14 days** (vibration off), weight 2.5–3.5 g
- Circular Ring 2: ~8 days Eco / ~4 days performance, **~5 g**, from $379 — the ECG outlier

So real titanium rings now span **~2–5.2 g**. The model's weight floor was lowered to
match the Oura Ring 5 era; ECG-equipped rings (Circular Ring 2) reach the ~5 g top end.

Capacity and weight figures above are **real**; the per-size battery weight/cost are
modeled (vendors don't publish cell-level weight or BOM cost).

---

## 5. The sensors

For each: what it is, what it does, what metrics it unlocks, its modeled marginal
**power draw** (mAh/day), **weight** (g), **cost** ($), dependencies, and the LED color
shown on the inner wall of the 3D ring (sensors that physically sit against the skin).

Power draws are **modeled estimates** anchored to component datasheets (see §8). Weights
and costs are **modeled** — not publicly disclosed by any ring maker.

### Core sensors (always on — every health ring has these)

#### Multi-wavelength PPG  *(core)*
- **What it is:** A cluster of green + red + infrared LEDs and matching photodiodes —
  photoplethysmography. The workhorse optical sensor; everything optical hangs off it.
- **Does:** Shines light into the finger tissue and reads the reflected pulse waveform.
- **Unlocks:** Heart rate, HRV (heart-rate variability), respiration rate.
- **Power:** 0.6 – 1.4 mAh/day  → the **dominant single sensor**. (LEDs pulse at high
  peak current but are heavily duty-cycled; the average is what matters.)
- **Power caveat (from audit):** The optical AFE itself is only ~10 µA at 25 sps
  (MAX86141 ≈ 11 µA dual-channel), but that **excludes the LED drive current**, which is
  mA-level when LEDs pulse (31–124 mA peak per LED), and rises to 660–978 µA at 4096 sps.
  So the real energy goes into the **LEDs, not the AFE** — the 0.6–1.4 mAh/day figure
  assumes heavy duty-cycling, which is why duty cycle is the true lever.
- **Weight:** 0.12 g  ·  **Cost:** $6 – $12
- **LED on ring:** green
- **Note:** Several "metrics" people think are separate sensors are actually computed
  from this one waveform — see SpO₂, BP trends below.

#### Skin temperature  *(core)*
- **What it is:** NTC thermistor cluster (negative-temperature-coefficient).
- **Does:** Tracks skin/body temperature trends.
- **Unlocks:** Body temperature, menstrual-cycle prediction, illness/recovery signals.
- **Power:** 0.02 – 0.08 mAh/day  → negligible.
- **Weight:** 0.03 g  ·  **Cost:** $1 – $3
- **LED on ring:** grey (marker only; it's a contact thermistor, not optical)

#### 3-axis accelerometer  *(core)*
- **What it is:** MEMS motion sensor.
- **Does:** Detects motion and orientation.
- **Unlocks:** Steps, sleep staging, restlessness, auto-workout detection, gesture input.
- **Power:** 0.02 – 0.3 mAh/day. Real datasheets: 0.2–14 µA depending on mode
  (e.g. ADXL362 ~0.3 µA wake / 2 µA active; ADXL367 0.89 µA; BMA400 1–14 µA).
- **Weight:** 0.03 g  ·  **Cost:** $1 – $3
- **LED on ring:** none (not an optical/skin sensor)

### Optional sensors

#### SpO₂ blood oxygen
- **What it is:** Not separate hardware — it **reuses the red + IR pathways of the PPG**.
- **Requires:** PPG (auto-enabled when you select this).
- **Does:** Extra overnight red/IR sampling to estimate blood-oxygen saturation.
- **Unlocks:** Blood oxygen (SpO₂).
- **Power:** 0.2 – 0.6 mAh/day (the marginal cost of the extra sampling).
- **Weight:** 0 g  ·  **Cost:** $0 (shares the PPG hardware)
- **LED on ring:** red

#### IR alignment sensor
- **What it is:** A dedicated infrared emitter/detector for ring orientation.
- **Requires:** PPG.
- **Does:** Detects when the ring has twisted and switches to a cleaner PPG LED/photodiode
  pair to keep the signal accurate.
- **Unlocks:** Signal-quality / auto-realignment (better data, no new metric).
- **Power:** 0.05 – 0.2 mAh/day.
- **Weight:** 0.04 g  ·  **Cost:** $2 – $5
- **LED on ring:** amber

#### EDA / electrodermal activity
- **What it is:** Skin-conductance electrodes for stress sensing. Newer to rings;
  vendors sometimes blur "EDA sensor" vs. "stress estimated from HRV."
- **Does:** Measures electrodermal activity.
- **Unlocks:** Stress, arousal.
- **Power:** 0.2 – 0.5 mAh/day.
- **Weight:** 0.08 g  ·  **Cost:** $4 – $10
- **LED on ring:** blue (marker)
- **Uncertainty flag:** higher than average — definition and presence vary by vendor.

#### BP / vascular trend
- **What it is:** Not separate hardware — a **cuff-calibrated vascular/blood-pressure
  *trend* computed from the PPG waveform**.
- **Requires:** PPG.
- **Does:** Reports a BP/vascular *trend* — **not a numeric reading**, and needs a cuff
  to calibrate.
- **Unlocks:** BP / vascular trend (not a reading).
- **Power:** 0.2 – 0.5 mAh/day (extra processing/sampling).
- **Weight:** 0 g  ·  **Cost:** $0 (shares PPG hardware)
- **LED on ring:** green
- **Caveat (from audit):** **Trend-only.** RingConn rebranded its feature to "Vascular
  Health Insights" (cuff-calibrated, not live at launch); Oura Ring 5 added "Blood
  Pressure Signals" (overnight trend, not a cuff replacement). The **only** ring with
  genuine BP medical-device clearance is **Sky Labs CART** (CE-MDR + UK MHRA, ISO
  81060-2; **US FDA still pending**). The difference between a "BP ring" and a basic Oura
  is algorithms + regulatory clearance, not the sensor.

#### ECG (single-lead)
- **What it is:** Finger electrodes on the **exterior** of the ring for a single-lead
  electrocardiogram. The one genuinely new sensor *type* in 2026 rings.
- **Does:** Records a single-lead ECG on demand; runs an AFib-detection algorithm.
- **Unlocks:** Single-lead ECG, AFib detection.
- **Power:** 0.05 – 0.3 mAh/day — readings are on-demand (~30 s each), so the daily
  average is small despite the active draw during a reading.
- **Weight:** 0.6 g  ·  **Cost:** $15 – $40 (premium; pushes the ring toward ~5 g)
- **LED on ring:** none (electrodes, not optical/skin-contact LEDs)
- **Reality check (from audit):** Only the **Circular Ring 2** ships finger-electrode ECG
  (FDA-cleared AFib via a B-Heart algorithm). It weighs **~5 g** and starts at **$379**
  — far heavier/pricier than the 2 g slim rings. ECG reliability has been questioned by
  reviewers. Oura and Galaxy Ring do **not** have ECG (despite occasional spec-chart
  confusion). ECG needs two electrodes across a current path, which is why most rings
  can't cleanly offer it.

#### Haptic actuator
- **What it is:** A small vibration motor. An **output**, not a sensor.
- **Does:** Silent buzz for alarms and alerts.
- **Unlocks:** Silent alarms.
- **Power:** 0.02 – 0.1 mAh/day — only draws power during an alert, so the daily average
  is tiny. (RingConn quotes meaningfully longer life with vibration *off*.)
- **Weight:** 0.12 g  ·  **Cost:** $3 – $8
- **LED on ring:** none

#### Capacitive touch
- **What it is:** A capacitive surface. An **input**, not a sensor.
- **Does:** Tap & swipe to dismiss alerts and control the ring.
- **Unlocks:** Touch controls.
- **Power:** 0.05 – 0.25 mAh/day.
- **Weight:** 0.04 g  ·  **Cost:** $2 – $6
- **LED on ring:** none
- **Pairs with:** the microphone's Push-to-hold mode (you hold this surface to talk).

#### NFC payments
- **What it is:** A passive contactless-payment chip. **No biometrics.**
- **Does:** Tap-to-pay.
- **Unlocks:** Contactless pay.
- **Power:** 0 – 0.02 mAh/day — effectively free; energized by the reader's RF field.
- **Weight:** 0.10 g  ·  **Cost:** $3 – $7
- **LED on ring:** none
- **Note:** Payment-only rings (e.g. McLear) are a different animal — NFC and nothing else.

---

## 6. Microphone (one card, three listening modes)

A single MEMS microphone, but **how it listens** changes the power cost enormously. This
card is the headline lesson of the whole tool.

- **What it is:** On-ring MEMS microphone.
- **Unlocks (base):** Voice notes, assistant.
- **Weight:** 0.10 g  ·  **Cost:** $6 – $14
- **LED on ring:** none (acoustic, not optical)

Exactly one mode is active at a time:

| Mode | What it does | Power (mAh/day) | Extra unlock | Notes |
|------|--------------|-----------------|--------------|-------|
| **Always-on listening** | Continuous capture | **5 – 12** | Always-on capture | Dominates the budget; can drop a build to ~1–2 days. Arguably impractical in a ring. |
| **8-hour daily window** | Listens during an 8-hour window | **1.5 – 4** | Scheduled listening | Middle ground. |
| **Push-to-hold** | Records only while you hold the touch surface | **0.1 – 0.4** | Push-to-talk | **Requires Capacitive touch** (auto-enabled). Cheapest by far. |

**Key nuance:** *wake-word* listening (just waiting for a trigger phrase) is genuinely
cheap — dedicated wake-word MEMS mics draw only **5–10 µA** (≈0.1–0.2 mAh/day). What's
expensive is **continuously capturing/streaming audio** and the processing/radio it
triggers. "Always-on listening" here models continuous capture, hence the large range.

---

## 7. The 3D ring model

- **Source file:** `TI RING HALF.STEP` — a SolidWorks STEP (AP203) CAD solid you provided
  (~24 mm diameter, ~7 mm band; despite the name it is a full ring, full 360° coverage).
- **Pipeline:** STEP → GLB via OpenCASCADE (`cascadio`), re-tessellated to ~15.7k faces,
  centered and normalized, then **base64-inlined** into the HTML so the page is a single
  double-clickable file.
- **Render:** three.js WebGL. Polished-titanium PBR material (high metalness, low
  roughness) lit by a generated studio environment map + key/rim/fill lights, on a gentle
  auto-spin with a soft contact shadow.
- **Sensor LEDs:** small emissive spheres seated on the **measured inner wall**
  (normalized radius 0.80, just inside the 0.824 inner surface; hole axis = X). They
  appear/disappear with selection and are colored per sensor (see each sensor above).
- **Requires internet** for the three.js CDN; offline it shows a graceful fallback.

---

## 8. What's REAL vs. MODELED

**Real / verifiable (cited in §9):**
- Battery capacities: 17 / 18.5 / 22.5 mAh by Galaxy Ring size.
- Real ring battery life: 5–14 days across Oura, Galaxy, RingConn.
- Real ring weight: 2.3–5.2 g titanium.
- Component-level current draws used to bound the ranges (accelerometer 0.2–14 µA,
  wake-word mic 5–10 µA, MAX86141 PPG 10 µA low-power, etc.).
- The ~1.5–3 mAh/day whole-system budget (derived from capacity ÷ battery life).

**Modeled / estimated (clearly labeled in the UI):**
- The split of the system budget into a "platform baseline" + per-sensor marginal draws.
- Each sensor's exact mAh/day range (anchored to datasheets, but the allocation is ours).
- All per-sensor **weights** and **costs** — no ring maker publishes a per-component BOM.
- The "Est. parts cost" total — order-of-magnitude only.

**Why no exact per-sensor numbers exist:** manufacturers don't release per-sensor power,
weight, or cost breakdowns. The honest move is ranges, anchored to real component
datasheets and the hard whole-system battery envelope — which is what this model does.

**Things the model deliberately does NOT claim (per the audit):**
- **No non-invasive glucose.** No ring measures blood glucose with medical accuracy; the
  FDA (Feb 21 2024) has authorized none. "Glucose" features (e.g. Oura, Ultrahuman) are
  integrations with a *separate inserted CGM*, not ring hardware.
- **ECG is rare, not standard.** Only the Circular Ring 2 ships it; it's modeled as an
  optional, heavy, premium add-on.
- **BP is trend-only.** No consumer ring gives cuff-accurate readings; only Sky Labs CART
  has medical-device clearance, and not in the US.
- The **1.5–3 mAh/day system budget is a derived engineering estimate** (capacity ÷ real
  battery life), corroborated by summed component currents — not a vendor-published spec.

---

## 9. Sources

- Galaxy Ring battery capacity by size — SamMobile:
  https://www.sammobile.com/news/samsung-galaxy-ring-battery-capacity-details/
- Galaxy Ring battery sizes/life — nextpit:
  https://www.nextpit.com/samsung-galaxy-ring-how-long-battery-size-life-capacity
- Oura Ring 4 specs (5–8 days, 3.3–5.2 g, titanium, 20–80 min charge):
  https://support.ouraring.com/hc/en-us/articles/33045011508115-Oura-Ring-4
- RingConn Gen 2 / Gen 3 (10–14 days, 2.5–3.5 g, 500 mAh case):
  https://ringconn.com/products/ringconn-gen-2
- Low-power MEMS accelerometers (ADXL362 / ADXL367 / BMA400 currents):
  https://www.analog.com/en/products/adxl362.html
- Always-on / wake-word MEMS microphone power (5–10 µA) — DigiKey:
  https://www.digikey.com/en/articles/how-to-reduce-power-consumption-in-always-on-voice-interface-designs
- MAX86141 optical PPG / pulse-ox front-end (datasheet):
  https://www.mouser.com/datasheet/2/256/MAX86140-1139682.pdf
