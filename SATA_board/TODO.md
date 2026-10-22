# SATA_board – TODO / open items

Schematic review status as of 2026-10-22. Footprints are not finalised yet.

## Open questions

- [ ] **Input current / J1:** what can the laptop VSYS rail supply on battery? At 6 V the board draws ~10 A at full load (up to ~16 A at the 5 V sheet minimum). Is battery mode normal use or only a short backup?
- [ ] **Corsair Type 5 pinout:** check J8/J9 against the actual cable with a meter. Corsair doesn't publish the PSU-side pinout, and a swap would put 12 V on drive 5 V pins. Also check that the cable mates with Micro-Fit 43045-0612.
- [ ] **HDD model:** 12 V spin-up peak vs the eFuse breaker (3.04 A, R = 499 Ω, ~2.2 ms ITIMER blanking, 6.4 A fast trip).

## Power stage – open

- [ ] **F1:** pick a real part. Needs roughly a 15–20 A slow-blow rating (footprint is now 2512; check a fuse that size exists at that rating, else use a Nano2-style part).
- [ ] **J1:** VIN is on a single Nano-Fit pin (~6 A max). Use more pins or a bigger connector for VIN/GND.
- [ ] **LM5116 sheet:** re-run step 11 with CSD17578Q5A gate charge to clear the VCC-current warning. Only then consider moving VCCX from 12 V to the 5 V output (watch the 15 mA startup limit and HB-SW ≥ 4.75 V).
- [ ] **LM5175 boost-mode stability:** crossover is 10.7 kHz but the boost RHP zero is ~9 kHz at 5 V input. The sheet only models buck mode, so check on the bench at low VIN and full load (load-step test).
- [ ] **Thermal:** ~5 W loss at 5 V in / 6 A out (Q1, L1, R3). Give them copper and vias.
- [ ] **Part selection still generic:**
  - L1 4.7 µH, Isat > 22 A, DCR ~6 mΩ (e.g. Coilcraft XAL1510-472; footprint exists in KiCad).
  - L2 3.3 µH, Isat ≥ 8.4 A (XAL6060/XAL7070-332 class).
  - R3 8 mΩ ≥ 2–3 W (2512 or parallel pair).
  - R35 13 mΩ ≥ 0.5 W.
  - C16–C22 and C1 rated ≥ 35 V, ideally 50 V (the SMAJ24A clamps ~39 V).
  - Ceramic output caps: check DC-bias derating at 5 V and 12 V.
- [ ] **LM5175 sheet label:** the buck-leg FET name box still says CSD18563Q5A. Values are correct (CSD18503Q5A/CSD18504Q5A); just tidy it.

## STM32 telemetry (WIP) – noticed so far

- [ ] **+3.3V has no source.** It currently has only U7, its decoupling caps and the pull-ups. Needs a regulator (from 5 V or 12 V).
- [ ] **VIN_ADC divider:** R40/R41 gives ÷8 (24 V → 3.0 V). A TVS clamp event (~39 V) would put ~4.9 V on PB0. Check pin tolerance or add a clamp/series resistor.
- [ ] **IMON scaling:** 1 kΩ gives 0.246 V/A (3 A → 0.74 V). Consider ~3.3 kΩ for more ADC range, staying within the TPS25980 VIMON(max) recommendation.
- [ ] **Fan TACH:** needs a pull-up to 3.3 V (or internal pull-up). Check that the PWM/TACH pins (PA15, PB3, PB4, PB5) are 5 V tolerant, since fans can pull PWM up to ~5 V internally.
- [ ] **SWD:** SWDIO/SWCLK go nowhere yet. Add a header (plus NRST, 3V3, GND).
- [ ] **NRST:** add ~100 nF to GND alongside SW1/R39.
- [ ] **Comms:** PA9/PA10 (UART) and PA11/PA12 (USB) are unconnected. Decide the telemetry link to the laptop.
- [ ] **Crystal Y1:** check the 12 pF load caps against the ABM3B load capacitance spec.
- [ ] **VDDA:** consider a dedicated 1 µF + 10 nF (optionally a ferrite) for ADC accuracy.

## ERC cleanup

- [ ] Add `PWR_FLAG` on VIN, VIN_F, GND, 5175_VCC, 5116_VCC, the HB net and +3.3V (once it has a source).
- [ ] Add no-connect flags on J8/J9 pin 3 and any unused STM32 pins.
- [ ] Add the LM5116 footprint library to `fp-lib-table` (ERC footprint_link warning).

## Done

- [x] TPS25980 symbol: hidden `power_in` "IN" pins merged +5V into +12V. Fixed with stacked pins `[1-3,16]` / `[17-24]`.
- [x] 5 V eFuses changed to TPS259803 (7.6 V OVLO).
- [x] LM5116 bootstrap diode added (D8, PMEG4010CEJ).
- [x] LM5175 VCC cap added (C60, 1 µF).
- [x] Q5 gate clamp (D9, BZV55B10).
- [x] LM5175 compensation set to sheet values (8.06k / 3.9 nF / 220 pF).
- [x] C14/C15 boot caps → 470 nF.
- [x] C52 → 2.2 µF.
- [x] dVdt caps → 1 nF.
- [x] R1 → 2.2 Ω.
- [x] C7 → 8.2 nF.
- [x] CSG Kelvin net via NT2.
- [x] LM5175 FETs: Q1 CSD18503Q5A, Q2 CSD18504Q5A, Q3/Q4 CSD16321Q5.
- [x] LM5116 FETs: Q8/Q9 CSD17578Q5A.
- [x] Boot/VIN diodes → PMEG4010CEJ.
- [x] Input: fuse F1, SMAJ24A TVS moved after the fuse onto VIN_F, C61 bulk polymer.
- [x] PGOOD → EN_A/EN_B isolated with D13 (BAT54C); PGOOD and eFuse PGs routed to STM32.
