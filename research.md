# RingForge Technical Accuracy Audit: Smart Ring Hardware Claims (June 2026)

> External technical audit of the RingForge data model and `context.md` claims, checked
> against primary sources (Samsung support pages, Analog Devices & Bosch datasheets,
> the MAX86141 datasheet, Oura/RingConn/Sky Labs material, and the FDA safety
> communication). Captured 2026-06-17. This is the source research; `context.md` is the
> living model doc and has been updated to reflect the confirmed corrections below.

## TL;DR
- Most of the document's hardware claims hold up well. The Samsung battery-capacity figures, component current draws, sensor architecture, NFC, microphone, and haptic claims are all CONFIRMED against primary sources (Samsung support pages, Analog Devices and Bosch datasheets, manufacturer specs).
- The biggest thing to fix is that the document is now outdated on flagship models: the Oura Ring 5 shipped June 4 2026 (6-9 days, from 2 g, $399 base), so any "Oura Ring 4 is the latest" framing should be updated, and RingConn Gen 3's blood-pressure claim has been softened to "vascular trends," with the feature not live at launch.
- The whole-system 1.5-3 mAh/day power envelope is physically sound and matches the component-level numbers, but treat it as a defensible engineering estimate rather than a vendor-published spec.

## Key Findings

### 1. Samsung Galaxy Ring battery capacities by size — CONFIRMED
The exact figures and size mapping are correct. Samsung's own support documentation and FCC filings (reported by SamMobile and nextpit) confirm:
- Sizes 5-7: 17 mAh
- Sizes 8-11: 18.5 mAh
- Sizes 12-13: 22.5 mAh

Samsung rates "up to 7 days" for the largest sizes and "up to 6 days" for the smaller ones. Source quality is excellent here: this is corroborated by Samsung.com support pages, SamMobile, and nextpit, all tracing back to the FCC certification. Keep these numbers exactly as written.

### 2. Whole-system power budget of 1.5-3 mAh/day — REASONABLE ESTIMATE (physically sound)
The reasoning is sound and the envelope is defensible. Shipping rings carry 17-22.5 mAh (Samsung) up to roughly 20 mAh-class cells and last 5-14 days. Working the math: 22.5 mAh / 7 days = 3.2 mAh/day; 18 mAh / 12 days = 1.5 mAh/day. RingConn Gen 3 reaching up to 14 days on a small cell implies an even lower daily draw at the bottom end. This is not a vendor-published number, so label it in RingForge as a derived estimate, not a spec. The component-level draws below independently support it: the always-on sensors (accelerometer at 1-3 µA, PPG heavily duty-cycled, MCU and BLE radio sleeping the vast majority of the time) sum to well under a few mAh/day.

### 3. Real-world battery life of shipping rings — MOSTLY CONFIRMED, one framing update
- **Oura Ring 4: CONFIRMED.** Oura officially states "typically 5-8 days," charging 20-80 minutes, all-titanium, weight 3.3-5.2 g (Oura support, NBC Select). All correct.
- **Samsung Galaxy Ring: CONFIRMED at 6-7 days.** Android Authority's hands-on found about 4-7 days real-world; Samsung claims up to 7 (the "up to 9" figure was an early pre-launch claim from MX head TM Roh and should not be used).
- **RingConn Gen 2: CONFIRMED at 10-12 days.**
- **RingConn Gen 3: CONFIRMED.** Up to 14 days with vibration off, 10-12 with vibration on; weight 2.5-3.5 g; 500 mAh universal charging case good for roughly 150 days / 15+ charges. Anchor the real-world number nearer 10 days: The Gadgeteer drained to 34% after 7 days of full use with alerts active (about 10 days projected), while a Lifehacker/Yahoo reviewer got 17 days. Treat 14 days as best-case (vibration off).

### 4. Titanium ring weights of 2.3-5.2 g — CONFIRMED, widen the low end
The range is accurate for the mainstream titanium rings. Oura Ring 4 is 3.3-5.2 g, Ultrahuman Ring Air 2.4-3.6 g, RingConn Gen 2/Gen 3 about 2-3.5 g, Galaxy Ring about 2.3-3.2 g. The new Oura Ring 5 (titanium) now goes as low as 2 g and tops out at 2.69 g (Tech Advisor hands-on: "it weighs from just 2g if you pick one of the smaller sizes. Even the largest Ring 5 only tips the scales at 2.69g"). Widen RingForge's range to roughly **2-5.2 g** for 2026. Note that the heavier, sensor-packed outliers like the Circular Ring 2 run about 5 g, so if you include ECG rings the top of the range stretches further.

### 5. Component-level current draws — CONFIRMED against datasheets
- **ADXL362:** Analog Devices datasheet says about 1.8 µA at 100 Hz, less than 3 µA across its full data-rate range, and 270 nA (0.27 µA) in motion-triggered wake-up mode. The document's "~0.3 µA wake / 2 µA active" is essentially correct (wake is 0.27 µA, active about 1.8-2 µA).
- **ADXL367: CONFIRMED** at 0.89 µA at 100 Hz ODR and 180 nA in wake-up mode (Analog Devices).
- **BMA400: CONFIRMED.** Bosch rates it at 14.5 µA max performance, 5.8 µA typical use, 3.5 µA low-power use case, 200 nA sleep. The document's "1-14 µA" range is accurate.
- **Always-on / wake-word MEMS microphone at 5-10 µA: CONFIRMED.** PUI Audio PMM-3738-VM1010 wake-on-sound draws 5 µA (9 µW); Vesper VA1210 wake-on-voice draws 10 µA. The 5-10 µA range is well-supported.
- **MAX86141 optical AFE at ~10 µA low-power mode: CONFIRMED.** The Analog Devices datasheet (Rev 5, 7/23) headline spec is "Optical Readout Channel <10 µA (typ) at 25 sps." Exact figures from the Electrical Characteristics table (LP_MODE=1, 25 sps): MAX86140 single channel 8.5 µA, MAX86141 dual channel 11 µA. **Add this caveat in RingForge:** this is the AFE supply current only and excludes the LED driver current, which is mA-level when LEDs pulse (e.g. 31-124 mA peak per LED); and ~10 µA only holds at low sample rates (25 sps) with dynamic power-down enabled. At the full 4096 sps the AFE alone draws 660 µA (MAX86140) to 978 µA (MAX86141). Shutdown current is 0.6 µA.

### 6. Multi-wavelength PPG is the dominant optical sensor — CONFIRMED
Green + red + IR LEDs with photodiodes is correct and standard. Green + IR measure heart rate, HRV, and respiration rate; red + IR measure SpO2. The Oura Ring 4 uses 10 LEDs across three wavelengths (green 520 nm, red 660 nm, IR 940 nm) forming 18 signal pathways; the Oura Ring 5 uses 12 pathways with stronger LEDs. Confirmed via Oura's own technical pages and independent teardown writeups.

### 7. SpO2 reuses the red + IR PPG pathways, not separate hardware — CONFIRMED
Correct. SpO2 is computed from the ratio of red (660 nm) to infrared (940 nm) absorption using the same PPG sensor and photodiodes. Both Oura's documentation and the MAX86141 datasheet describe SpO2 as using red and IR LEDs that share the same photodiode and separation distance. No separate hardware.

### 8. Cuffless blood pressure is trend-only, algorithm/regulatory-gated — CONFIRMED with important updates
The core claim is correct: pulse-wave BP from rings is trend-only, computed from the PPG waveform, and the difference between a BP ring and a basic Oura is algorithms plus regulatory clearance, not fundamentally different hardware. Updates:
- **Sky Labs CART is the real regulated outlier.** CART BP Pro obtained CE-MDR certification in Europe in January 2026 and completed UK MHRA medical-device registration on May 15 2026; it met ISO 81060-2:2018. It is MFDS-approved in South Korea and, per Sky Labs' January 2026 release, "has been utilized in approximately 1,700 hospitals and clinics across Korea" since reimbursement in 2024, with cumulative prescriptions "surpassing approximately 260,000 cases as of May 2026." Sky Labs describes it as "the world's only cuffless ring-type blood pressure monitor" with this level of clearance. **US FDA clearance is still in progress, not granted.** So a ring-type cuffless BP monitor with genuine medical-device clearance now exists, but not yet in the US.
- **RingConn Gen 3:** BP was rebranded from "Blood Pressure Insights" to "Vascular Health Insights / Vascular Trend." It needs a cuff to calibrate, reports trends not single numeric readings, and was NOT live at launch (arriving via software update). RingConn explicitly states it is a health-management device, not a medical device.
- **Oura Ring 5** added "Blood Pressure Signals" (overnight trend detection), also explicitly not a cuff replacement, and lets users log cuff readings manually.

### 9. EDA / electrodermal activity in rings — CONFIRMED, definition varies by vendor
The claim is accurate. Genuine EDA-electrode rings exist but are niche: the Moodmetric ring and the BIOPAC Research Ring use real skin-conductance electrodes, and Happy Health's Happy Ring shipped a custom EDA sensor. The mainstream consumer rings (Oura, Galaxy Ring, RingConn, Ultrahuman) compute "stress" from HRV and temperature, not from a dedicated EDA electrode. So vendors do blur "real EDA sensor" versus "stress estimated from HRV," exactly as the document says. Flag this clearly in RingForge so EDA is not assumed present just because a ring reports a stress metric.

### 10. Skin temperature via NTC thermistor — CONFIRMED
Correct. Rings use an NTC (negative temperature coefficient) thermistor on the inner surface to track skin temperature, reported as deviation from a personal nightly baseline rather than absolute temperature. This unlocks illness/recovery signals and menstrual-cycle prediction. The UCSF TemPredict study used Oura temperature data for early COVID detection. Confirmed via Oura and RingConn technical pages. One nuance worth noting: Oura's sensor is described as an NTC thermistor that does not rely on direct skin contact, reading near-skin temperature.

### 11. NFC payments are passive, reader-powered, zero daily draw — CONFIRMED, with caveat
Correct in principle: passive NFC payment chips have no battery, are energized by the reader's RF field, and add effectively zero daily power. McLear RingPay is the canonical payment-only ring (zirconia ceramic, no charging required). Two important caveats to add: (1) McLear's RingPay program closed to new general sales as of 31 October 2024, so it is a fading example rather than a current product; and (2) metal (titanium) shields NFC, which is part of why the titanium health rings do not include payments and why payment rings use ceramic.

### 12. Microphone power nuance — CONFIRMED, and continuous listening is impractical in a ring
The distinction is technically sound. Wake-word / voice-activity detection is cheap (5-10 µA, roughly 0.1-0.2 mAh/day). Continuously capturing and streaming audio plus the DSP and BLE radio it wakes is far more expensive, and modeling that at 5-12 mAh/day is reasonable. **Assessment: an always-on continuously-listening microphone is not practical in a current smart ring.** On a 17-22.5 mAh cell, 5-12 mAh/day of audio load would cut battery life to 1-2 days or less, defeating the entire multi-day value proposition that defines the category. Wake-word standby is fine; continuous streaming plus its triggered processing is not. This is a good guardrail to bake into RingForge: let users toggle a "wake-word mic" cheaply, but warn hard against "always streaming."

### 13. Haptic actuator — CONFIRMED
Correct. A small vibration motor draws power only during alerts, so daily average is tiny. RingConn Gen 3 is the proof point: it explicitly quotes longer battery life with vibration off (11-14 days) versus on (10-12 days), based on about 20 seconds of buzzing per day. More buzzing drains faster. Confirmed via RingConn and multiple independent reviews.

### 14. Other 2026 developments RingForge should add
- **Oura Ring 5 (shipped June 4 2026):** Per Oura's May 28 2026 announcement, "the smallest smart ring in the world," 40% smaller than Ring 4, 6.09 mm wide, 2.28 mm thick, from 2 g (max 2.69 g), 6-9 day battery, redesigned sensors using 12 signal pathways (down from 18) with roughly 4x stronger LEDs, new physical-vapor-deposition scratch coating, "$399 for base finishes in Silver and Black" / $499 premium. No ECG, no NFC, no GPS. Most new software (Health Radar, Blood Pressure Signals, Nighttime Breathing) also rolls out to Gen 3 and Ring 4.
- **Galaxy Ring 2: NOT released.** Delayed to likely early 2027 amid an Oura patent dispute (Oura filed against Samsung in late November 2025). Rumored focus on battery (9-10 days), comfort, sensor accuracy, possible onboard temperature sensor, and longer-term non-invasive glucose ambitions.
- **Circular Ring 2 (shipping mid-2025):** The notable new sensor type — first ring with a single-lead ECG and FDA-cleared AFib detection (via a B-Heart algorithm) using finger electrodes on the outside of the ring. Correct the document here: it weighs about **5 g** (Circular CEO Amaury Kosman: it "weighs about 5g, well above the 2g Slim"), not 3 g, and it starts at **$379** (matte black), rising to $449 silver and $549 gold/rose gold — not the ~$349-380 sometimes cited. Battery is about 8 days in Eco mode, 4 days in performance mode. ECG reliability is questioned by reviewers: Tom's Guide's reviewer reported "despite numerous attempts, I wasn't able to get the feature to work. Each time I took a reading, the results said my ECG couldn't be analyzed."
- **Ultrahuman Ring Air:** 2.4-3.6 g, PPG + non-contact skin-temp sensor + 6-axis IMU, no subscription. Its "glucose" is CGM integration that pairs with a separate inserted biosensor, not ring-based glucose. Still tied up in US patent litigation with Oura (availability is hit-or-miss in the US).
- **Glucose:** No ring measures blood glucose non-invasively with medical accuracy. Per the FDA Safety Communication (Feb 21 2024): "The FDA has not authorized, cleared, or approved any smartwatch or smart ring that is intended to measure or estimate blood glucose values on its own." Consumer "glucose rings" (RIZZ, Aurora, etc.) are wellness-trend estimates only (70-85% accuracy in ideal conditions). Oura's glucose feature integrates Dexcom's separate inserted Stelo CGM, not ring hardware.
- **ECG in rings:** Only the Circular Ring 2 ships finger-electrode ECG. Oura and Galaxy Ring do not have ECG, despite occasional marketing-chart confusion (one NBC spec line erroneously listed the Oura Ring 4 with ECG; Oura's own documentation lists no ECG).

## Recommendations
1. **Keep as-is:** the Samsung battery table, all component current draws, the PPG/SpO2/temperature sensor architecture, NFC, haptic, and microphone-nuance claims. They are all confirmed against primary sources.
2. **Update the flagship lineup:** add Oura Ring 5 as the current Oura (June 2026), keep Ring 4 as still-sold. Note Galaxy Ring 2 is not out (early 2027 expected). Add Circular Ring 2 as the ECG outlier and Sky Labs CART as the regulated BP outlier.
3. **Fix the BP module language:** change any "blood pressure reading" wording to "blood pressure / vascular trend (cuff-calibrated, not a reading)" for consumer rings, and flag Sky Labs CART as the only ring with real BP medical-device clearance (CE-MDR + UK MHRA, not US FDA yet).
4. **Add the MAX86141 caveat:** ~10 µA is AFE-only at 25 sps with low-power mode, excluding LED drive current (mA-level when pulsing) and rising to 660-978 µA at high sample rates. This makes the power model honest about where the real optical-sensor energy goes (the LEDs, not the AFE).
5. **Correct the Circular Ring 2 spec** if it is in RingForge's database: about 5 g and from $379, not 3 g / ~$350.
6. **Widen the titanium weight range** to about 2-5.2 g (2 g for Oura Ring 5; up to ~5 g if ECG rings are included).
7. **Label the 1.5-3 mAh/day system budget** explicitly as a derived engineering estimate, with the component-current sum as supporting evidence, not a vendor spec.
8. **Benchmarks that would change these recommendations:** a shipping ring with always-on audio that still clears 4+ days (would change claim 12); US FDA clearance of any cuffless BP ring (would change claim 8); any mainstream ring adding a true EDA electrode or non-invasive glucose with regulatory clearance (would change claims 9 and 14).

## Caveats
- Several battery and weight figures are manufacturer best-case lab numbers; real-world results run lower, so RingForge should model a range, not a point estimate.
- Some sources used here are secondary (review blogs, retailer pages). The strongest claims trace to Samsung support pages, Oura support, Bosch and Analog Devices datasheets, the MAX86141 datasheet (Rev 5), the FDA safety communication, and Sky Labs/Oura regulatory press releases.
- The smart-ring market is in active patent litigation (Oura vs Samsung, Ultrahuman, RingConn, Circular and others), so US availability of specific models is volatile and should be re-checked before RingForge surfaces a model as "buyable."
- Component current draws are datasheet typicals at 25 C and specific supply voltages; real duty-cycled in-ring behavior depends on firmware and sampling cadence, so the configurator should expose duty cycle as a tunable input rather than assuming continuous operation.
