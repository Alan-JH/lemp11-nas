# SATA_board – TODO / open items

Schematic review status as of 2026-10-22 (rev v9). Footprints are not finalised yet.

## Open questions

- [ ] **C28 (100 µF, EN_B delay) type:** must be a low-leakage ceramic (X5R/X7R, ≥ 6.3 V), not an electrolytic. The EN_B divider only has ~7 µA of margin before leakage keeps EN_B below the 1.2 V threshold forever (final 2.39 V, Thevenin 160 kΩ). A 6.3 V electrolytic is spec'd up to ~6 µA leakage. With a ceramic, expect ~6–11 s (DC-bias and ±20% tolerance) instead of the ideal 11.2 s.
- [ ] **Measure the spin-up profile** with IMON logging (1 kHz) to confirm the 3–8 s high-current phase on the WD100EDAZ.
- [ ] **Firmware:** PB6 (DIS_A) / PB7 (DIS_B) high = branch off. Releasing DIS_B restarts the full ~11 s RC delay (Q12 discharges C28), so the MCU can only add delay, never shorten it.
- [ ] **Battery power test (optional):** tested to ~35 W with no trip. With a proper stagger, the worst case (second drive spinning up while the first is busy) is ~36 W out / ~39–40 W in for a few seconds. Testing to ~45 W for 15–20 s would cover it with margin.

## Power stage – open

- [ ] **F1 → Bourns SF-1206S1000W-2** (10 A slow-blow, 32 VDC, 50 A interrupting, 11.3 A²s, 5 mΩ). Change the footprint from 2512 to 1206 and give both pads wide copper pours or vias (a 10 A 1206 relies on board copper for its rating).
- [ ] **LM5116 sheet:** re-run step 11 with CSD17578Q5A gate charge to clear the VCC-current warning. Only then consider moving VCCX from 12 V to the 5 V output (watch the 15 mA startup limit and HB-SW ≥ 4.75 V).
- [ ] **LM5175 boost-mode stability (lower priority now):** the ~9 kHz RHP zero was for 5 V in / 6 A out. At the real load (6 V in, ~3 A out) it's ~34 kHz, well above the 10.7 kHz crossover. Still do a quick load-step test on the bench at 6 V in.
- [ ] **Thermal:** at ~40 W the converter loss is ~3 W and only for seconds; ~1.5 W steady. Normal copper and vias under Q1/Q2/L1 should do.
- [ ] **Part selection still generic:**
  - L1 4.7 µH, Isat > 22 A, DCR ~6 mΩ (e.g. Coilcraft XAL1510-472). Isat is set by the LM5175 current limit (170 mV ±30% / R3 → up to ~27 A peak with 8 mΩ), not by the load, so it stays even at 40 W. R3 could go to ~10 mΩ (limit 12–22 A) since the real peak inductor current is ~9 A.
  - L2 3.3 µH, Isat ≥ 8.4 A (XAL6060/XAL7070-332 class).
  - R3 8 mΩ: ~0.4 W at the real ~7 A, so a 1 W 2512 is enough.
  - R35 13 mΩ ≥ 0.5 W.
  - C16–C22 and C1 rated ≥ 35 V, ideally 50 V. This is for MLCC DC-bias derating at 19–20 V, not the TVS clamp.
- [ ] **Hot-plug ringing on J1 (bench check):** if J1 is plugged in live, cable inductance with the input caps can ring well above 19 V. That's the one non-ESD event that could approach the TVS. The polymer bulk caps should damp it; scope VIN_F on a live plug-in once.
  - Ceramic output caps: check DC-bias derating at 5 V and 12 V.
- [ ] **LM5175 sheet label:** the buck-leg FET name box still says CSD18563Q5A. Values are correct (CSD18503Q5A/CSD18504Q5A); just tidy it.

## 3.3 V supply (U9 LT3080 → D14 OR → U10 TLV75533) – open

- [ ] **C71 voltage rating:** now on VIN_F, so ≥ 35–50 V like the other input ceramics.

## STM32 telemetry (WIP) – open

- [ ] **Fan PWM firmware:** Q7/Q11 invert the PWM (MCU high → fan PWM low). Invert the duty cycle in firmware. 3.3 V loss or MCU reset → fans run full speed (intended).
- [ ] **IMON accuracy:** the TPS25980 gain (246 µA/A ±7%) is only specified for IOUT ≥ 3 A. Normal loads are below that, so treat readings as approximate or calibrate each channel on the bench. Use a moderate ADC sampling time for the 10 kΩ source.
- [ ] **U8 USBLC6 layout:** it's wired flow-through (connector → pin 6/4, pin 1/3 → MCU). Pins 1/6 and 3/4 are the same net inside, so route the line straight across the pads with U8 close to J2.
- [ ] **VIN_ADC clamp leakage:** D12 (BAT54J to 3.3 V) reverse leakage rises with temperature. ~10 µA hot into the 1.3 kΩ divider source ≈ 0.1 V error at VIN scale. Fine for telemetry; use a low-leakage diode if better accuracy is needed.
- [ ] **USB series R (optional):** R47/R48 10 Ω are harmless but not needed on G4 FS USB; could be 0 Ω.
- [ ] **Crystal Y1:** check the 12 pF load caps against the ABM3B load capacitance spec.
- [ ] **VDDA:** VDDA is tied straight to +3.3V. Consider a dedicated 1 µF + 10 nF (optionally a ferrite) for ADC accuracy.
- [ ] **PB4 dead-battery pull-down:** PA10 is tied to GND, so the UCPD 5.1 kΩ pull-down on PB4 stays off. Also set `UCPD1_DBDIS` in `PWR_CR3` in firmware as a belt-and-braces.

## Footprints / layout (WIP) – open

- [ ] **In2 has no plane under the LM5175 power stage.** The In2 +12V zone only covers x ≈ 76–136 mm, so the bottom-side FETs (Q1–Q4, R3, D2/D3) see In1 GND ~1.3 mm away through the core. Fill In2 with GND under the LM5175 stage (and ideally GND everywhere In2 isn't +12V) so the bottom hot loops have a reference ~0.2 mm away.
- [ ] **Local HF caps on the bottom at the half-bridges.** The input/output ceramics are on top and the FETs on the bottom, so every hot loop goes through vias. Add 2–3 small caps (100 nF + 1 µF, 0603/0805) on the bottom right at Q1 drain ↔ Q2/R3 return (VIN_F side), and at Q3 drain ↔ Q4/R3 return (+12V side). Same idea for Q8/Q9 on +12V.
- [ ] **Zones are stale:** refill (B). The nRST and 5V_USB vias show 0 mm clearance to the In1 GND zone.
- [ ] **194 unconnected items**, mostly GND (stitching/plane vias not placed yet).
- [ ] **TPS25980 footprint clearance:** pad-to-pad 0.142/0.162 mm vs the 0.2 mm board rule (U3–U6, 22 each). Check against TI's land pattern and the fab minimum, then add a footprint-level clearance override or custom rule instead of ignoring the errors.
- [ ] **Mounting holes H1–H4:** copper too close to the board edge.
- [ ] **R32/C50 courtyard overlap.**
- [ ] **Bottom signal bundle near Q8/Q9 and L2:** check which nets these are. Keep IMON/ADC/PG lines well away from SW nodes; fan lines are fine.
- [ ] **C61, C8–C11 PCM1K680MCL1GS:** the 680 code normally means 68 µF, not 680 µF. Confirm the value and that the case matches CP_Elec_10x10.
- [ ] **C29/C32/C81/C82 (10 µF 0805 on +12V):** expect ~3–4 µF effective at 12 V DC bias. Fine if they're local decoupling; use 1206/1210 if you're counting on the capacitance.
- [ ] **L1 IHLP-6767 footprint:** pick the exact IHLP-6767 part number with Isat > 22 A and confirm the land pattern matches.

## ERC cleanup

- [ ] Add `PWR_FLAG` on VIN, VIN_F, GND, 5175_VCC, 5116_VCC, the HB net, +5V, 4V5_STBY and the U10 IN node.
- [ ] Add no-connect flags on J8/J9 pin 3, J2 SBU1/SBU2, and unused STM32 pins (PA9).

## Done

- [x] D16 (BAT54J) adds board +5V to the 3.3 V OR, so the LT3080 is idle whenever the converters are up.
- [x] LM5175 gate resistors R71–R74 = 3.3 Ω (0402) at the FET gates (≤ 3.3 Ω keeps the 55 ns dead time safe).
- [x] U9 minimum load: 8.2 kΩ on 4V5_STBY (0.52 mA vs 0.5 mA min; 6.8 kΩ gives more margin if needed).
- [x] 3.3 V rework: U9 LT3080 from VIN_F at 4.3 V (R54 430 kΩ) → D14 OR with VBUS → U10 TLV75533 (EN tied to IN, 10 µF in/out). LT3080 idle when USB is present; USB-only gives a solid 3.3 V (TLV755 dropout ≤ 238 mV at 500 mA, ~25 mV at 50 mA).
- [x] PB8-BOOT0 10 kΩ pull-down (R65).
- [x] USB ESD: USBLC6-2SC6 (U8) on D+/D−.
- [x] VIN_ADC clamp: BAT54J to 3.3 V (D12) and GND (D15).
- [x] Drive stagger: C28 → 100 µF (~11 s EN_B delay), plus MCU override Q12/Q13 (PB7/PB6, 10 kΩ gate pull-downs, so default = on if the MCU is dead).
- [x] Sizing basis: real max load ~40 W (staggered spin-up), not the converter ratings. J1 (2 VIN pins) is fine at ~7 A.
- [x] Corsair Type 5 pinout checked (user, from the cable).
- [x] HDD WD100EDAZ: 12 V peak ~1.79 A (WD Red 10 TB spec as a stand-in; WD doesn't publish white-label specs). The eFuse limit is 3.04 A typ, ~2.5 A worst-case min, so there's margin.
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
- [x] SWD header J3 (2×3) with 100 Ω series resistors; NRST 100 nF (C73).
- [x] USB-C J2 terminated correctly: separate 5.1 kΩ on CC1/CC2, D+/D− pairs tied, VBUS TVS (SMF6V0A), 10 µF on VBUS, SBU NC.
- [x] J1 → 2×3 Nano-Fit, VIN on 2 pins.
- [x] 3.3 V now powered whenever VIN or USB is present (LT3080 fed via D14 from VIN_F / VBUS). This fixes PGOOD/EN being held low with no USB and fans/IMON back-feeding an unpowered MCU. (LM1117 U8 removed.)
- [x] Fan PWM driven open-drain via 2N7002 (Q7/Q11, 100 Ω gate, 10 kΩ pull-down). No 5 V tolerance issue on PA15/PB3.
- [x] Fan TACH pull-ups R58/R59 10 kΩ to 3.3 V on PB5/PB4.
- [x] PA10 tied to GND (keeps PB4 UCPD dead-battery pull-down off).
- [x] VIN-side voltage ratings reviewed against the ~20 V operating max, not the 39 V TVS clamp (ESD energy is absorbed by ~750 µF of input capacitance). D14 BAT54C (30 V) is OK. VIN_ADC ÷8 is OK (20 V → 2.5 V; 26.7 V TVS breakdown → 3.3 V; PB0 abs max 4.0 V).
- [x] IMON: RIMON → 1.5 kΩ (0.37 V/A; worst-case fast trip ~7.5 A → 2.96 V, under the 3.3 V limit), plus 10 kΩ / 100 nF RC filter into PA0–PA3 (τ ≈ 1 ms), per TI's recommendation.
- [x] Footprint libraries resolved (U2 → LM5116 lib, Y1 → KiCad Crystal lib); ERC footprint_link warnings gone.
