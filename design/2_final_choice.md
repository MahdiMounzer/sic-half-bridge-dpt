**MOSFET Selection Study — Final Comparison**

**Three-way SiC candidate comparison & device selection**

SiC Half-Bridge Inverter Leg · DPT Characterization — Stage 0 Design Study

## **Scope & method**

Three discrete 650 V SiC MOSFETs in TO-247-4 (Kelvin source) packages are compared for the half-bridge inverter leg. Each was analysed in its own device document (reading → derivation → loss summary). This document consolidates the results and makes the selection.

All three are evaluated under identical assumptions so the comparison isolates the device: 200 V bus, I_peak = 10 A, hard-switched duty D = 0.5 (I_rms = 7.07 A), junction temperature 175 °C (worst case), switching loss reported at 50 and 100 kHz. Conduction loss uses R_ds(on) at 175 °C; switching loss uses datasheet E_on/E_off rescaled to 200 V / 10 A (E ∝ V·I), all at 175 °C.

# **1 · Candidates**

| **Manufacturer** | **Part number**  | **Technology** | **R_ds(on) 25°C** | **Package** |
| ---------------- | ---------------- | -------------- | ----------------- | ----------- |
| Infineon         | IMZA65R048M1H    | CoolSiC M1     | 48 mΩ             | TO-247-4 ✓  |
| Wolfspeed        | C3M0045065K      | C3M (Gen3)     | 45 mΩ             | TO-247-4 ✓  |
| onsemi           | NTH4LN032N065M3S | EliteSiC M3S   | 32 mΩ             | TO-247-4L ✓ |

_All three pass the hard filters: genuine silicon carbide, 650 V class, TO-247-4 with a Kelvin (driver-source) pin, and V_GS windows compatible with the TI UCC21520 driver. The onsemi part is specified at an 18 V gate drive (the others at 15 V); the UCC21520 supports both._

# **2 · Loss comparison**

_All values at 200 V / 10 A / D = 0.5 / 175 °C. Lowest value in each row is the best performer._

## **2.1 · Conduction loss (frequency-independent)**

| **Quantity**     | **Infineon** | **Wolfspeed** | **onsemi** | **Unit** |
| ---------------- | ------------ | ------------- | ---------- | -------- |
| R_ds(on) @175 °C | 67           | 61            | 49         | mΩ       |
| I_rms (D = 0.5)  | 7.07         | 7.07          | 7.07       | A        |
| **P_cond**       | 3.35         | 3.05          | **2.45**   | W        |

## **2.2 · Switching loss (per device)**

| **Quantity**              | **Infineon** | **Wolfspeed** | **onsemi** | **Unit** |
| ------------------------- | ------------ | ------------- | ---------- | -------- |
| E_on + E_off (200 V/10 A) | 25.6         | 20.2          | 18.6       | µJ       |
| **P_sw @ 50 kHz**         | 1.28         | 1.01          | **0.93**   | W        |
| **P_sw @ 100 kHz**        | 2.56         | 2.02          | **1.86**   | W        |

## **2.3 · Total loss — P_tot = P_cond + P_sw**

| **P_tot**     | **Infineon** | **Wolfspeed** | **onsemi** | **Unit** |
| ------------- | ------------ | ------------- | ---------- | -------- |
| **@ 50 kHz**  | 4.63         | 4.06          | **3.38**   | W        |
| **@ 100 kHz** | 5.91         | 5.07          | **4.31**   | W        |

**Ranking on total loss (both frequencies):** onsemi < Wolfspeed < Infineon. The onsemi 32 mΩ die wins on both loss families — its lower R_ds(on) drives conduction down, and it also carries the lowest switching loss even after the 175 °C correction.

# **3 · Frequency dependence**

The two loss families behave oppositely with frequency: P_cond is constant, while P_sw rises linearly with f_sw. This shifts which family dominates and therefore what the comparison rewards.

- At 50 kHz, conduction dominates every candidate (e.g. onsemi 2.45 W conduction vs 0.93 W switching) — the ranking is driven mainly by R_ds(on).
- At 100 kHz, switching loss roughly doubles and closes part of the gap, but does not change the order here — onsemi's conduction advantage is large enough to hold the lead.

_Because the design targets the 50–100 kHz range, the 100 kHz worst case is used for thermal sizing. The ranking is stable across the range, so the selection does not depend on the exact operating frequency._

# **4 · Secondary axes (tie-breaks & robustness)**

| **Axis**       | **Infineon** | **Wolfspeed** | **onsemi** | **Reads as**                         |
| -------------- | ------------ | ------------- | ---------- | ------------------------------------ |
| Body diode V_F | 4.0 V        | 4.8 V         | 4.5 V      | Dead-time conduction; Infineon best  |
| Q_rr           | 125 nC\*     | 171 nC        | 72 nC      | Recovery loss/ringing; onsemi best   |
| C_oss (max)    | 168 pF       | 101 pF        | 114 pF     | Overshoot/ringing; Wolfspeed best    |
| Q_g            | 33 nC        | 63 nC         | 55 nC      | Driver load; Infineon lightest       |
| Gate drive     | 15 V         | 15 V          | 18 V       | onsemi needs 18 V for rated R_ds(on) |

_\* Infineon lists Q_fr (forward recovery, includes Q_oss); classical Q_rr is near-zero for all three SiC parts. All Q_rr/Q_fr values are small — the SiC advantage over silicon (where this figure would be thousands of nC) holds across the board._

No secondary axis overturns the loss ranking. onsemi has the lowest Q_rr; Wolfspeed the lowest C_oss; Infineon the lightest gate charge and lowest V_F. These are second-order next to the ~1.6 W P_tot spread at 100 kHz.

# **5 · Selection & rationale**

**Primary pick: onsemi NTH4LN032N065M3S.**

It gives the lowest total loss at both frequencies (4.31 W vs 5.07 / 5.91 W at 100 kHz), the lowest reverse-recovery charge, and low output capacitance. The margin comes chiefly from its 32 mΩ die, which suppresses the conduction loss that dominates at these frequencies.

**Honest qualifications (the engineering reading behind the raw numbers):**

- Lower-resistance die. The onsemi part is 32 mΩ vs 45–48 mΩ for the others — not a perfectly like-for-like die, so some of its advantage is that it is simply a larger/lower-R device. Fair for a best-performer choice, but worth stating.
- 18 V gate drive required. Its rated R_ds(on) assumes an 18 V drive. The UCC21520 supports this, but the gate supply must be designed for +18 V (and a negative off-rail), not the +15 V the other two use.
- Temperature-corrected switching. Its switching loss was recomputed at 175 °C (E_off nearly doubles vs 25 °C). Without that correction the comparison would have flattered onsemi further; the ranking holds even after it.

**Cleanest like-for-like: Infineon vs Wolfspeed.**

These two are near-identical dies (48 vs 45 mΩ), both 15 V drive, both effectively hot data — the most directly comparable pair. Wolfspeed wins that head-to-head on every loss axis (P_tot 5.07 vs 5.91 W at 100 kHz), making it the natural second choice and a strong fallback if a 15 V-only gate scheme is preferred.

## **Estimate status & next step**

These P_tot figures are first-order estimates: switching energy from datasheet E_on/E_off rescaled linearly, and a simplified D = 0.5 hard-switched conduction condition. They are sufficient to rank the three candidates, and the ranking is robust across the 50–100 kHz range. The selected device proceeds to: (1) the Double Pulse Test, which replaces the estimated E_on/E_off with measured values at the true operating point, and (2) a realistic sinusoidal-inverter thermal-validation model using that measured data to confirm T_j stays within the 175 °C limit on the chosen heatsink.

**Decision**

**Select onsemi NTH4LN032N065M3S (lowest P_tot, 18 V drive).** Fallback: Wolfspeed C3M0045065K (best 15 V-drive option, cleanest peer to the Infineon baseline). Confirm with DPT-measured switching energies before committing to fabrication.