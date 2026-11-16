# SATA_board – Bench testing log

## 2026-11-05 – LM5175 (U1) first power-on

### Bring-up fault: no output

**Setup:** 19 V on VIN, 3.3 V applied to 3V3_M2 (enables U1 through Q6/Q5), no load.

**Symptoms:**
- No switching on SW1 and no output on +12V.
- 5175_VCC = ~7.4 V (at C60). Normal; the datasheet typical is 7.35 V.
- SS = ~1.2 V (at C6). The SS clamp is 1.27 V, so the controller was enabled and had finished soft-start (SS is held low in standby, VCC UV, hiccup and thermal shutdown).
- LDRV1 (probed at R73) switched for ~1.6 ms after power-on, then stopped. The high side (HDRV1/Q1) never switched, so SW1 stayed near 0 V.

**Cause:** a diode was fitted backwards.

**Fix:** replaced the diode with the correct orientation. The converter now regulates.

**Notes:**
- Schematic and PCB nets for U1 were checked against the LM5175 RHF pinout and match. EN/UVLO divider, MODE strap (R4 to GND, R5 DNP, so DCM with hiccup disabled), FB divider (12 V) and the bootstrap/gate-drive copper are all correct.
- If the high side doesn't switch, check BOOT1–SW1 (needs > 2.73 V for HDRV1 to run, ~7 V expected) and COMP before suspecting overcurrent. With MODE at GND, hiccup is disabled, so an overcurrent would not stop switching.

### SW2 (boost-leg) ringing, boost mode

Q3/Q4 are CSD16321Q5 (25 V). Snubber R75/C83 is not fitted (DNP).

| Input | Load on +12V | Probe ground | SW2 peak | % of 25 V |
|---|---|---|---|---|
| 7 V | none | 15 cm lead | 18–19 V | ~76% |
| 6 V | none | 15 cm lead | 19.2 V | 77% |
| 6 V | none | ground spring | 18.0 V | 72% |
| 6 V | 3–4 A | ground spring | < 18.0 V (small spike returns) | < 72% |
| 5.5 V | 3 A | ground spring | 15.6 V | 62% |

**Observations:**
- Ringing is worst at no load: a leading-edge spike that goes away under load. This fits light-load (DCM) operation, where Q3 is off at Q4 turn-off and the current briefly flows through Q3's body diode and D3.
- A small spike returns at 3–4 A (hot-loop overshoot, which grows with switch current), but it stays below the no-load peak.
- The 15 cm ground lead read ~1.2 V high compared with the ground spring.

**Result:** worst-case SW2 peak is 18.0 V (72% of the 25 V rating, under the ~80% / 20 V guideline). The snubber is not needed; leave R75/C83 unfitted. Fit and tune them only if a later change pushes the peak above ~20 V.

## 2026-11-06 – Drive stagger and spin-up

### EN_A → EN_B load-switch delay

- Delay between the two load-switch pairs turning on: **~10.92 s** (measured on the oscilloscope).
- Design expectation: 11.2 s ideal, ~6–11 s allowing for C28 DC bias and ±20% tolerance (see TODO.md). The result is close to ideal.

### Two-drive spin-up

- Two drives spun up successfully, staggered by the EN_B delay.
- Peak input current was **~3 A at 6 V input** (~18 W in), measured on the input during spin-up.

### Still to test

- [ ] Boost-mode load-step stability at 6 V input (from TODO.md).
