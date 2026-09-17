MOSFET Selection Study — Device Analysis

onsemi NTH4LN032N065M3S

_EliteSiC™ M3S — 650 V SiC MOSFET · TO-247-4L (Kelvin source)_

SiC Half-Bridge Inverter Leg · DPT Characterization — Stage 0 Design Study

## **Application operating point (cahier des charges)**

| **Parameter**              | **Value**           |
| -------------------------- | ------------------- |
| DC bus voltage (V_DS test) | 200 V               |
| Test current (I_D)         | 10 A                |
| Switching frequency        | 50 – 100 kHz        |
| Gate driver                | TI UCC21520         |
| Design junction temp (T_j) | 175 °C (worst case) |

# **1 · Datasheet parameters (reading only)**

_Values read directly from the NTH4LN032N065M3S datasheet (Rev 2), source noted. No calculation at this stage._

| **Parameter**               | **Symbol** | **Value**                      | **Unit** | **Source**      |
| --------------------------- | ---------- | ------------------------------ | -------- | --------------- |
| **Package / Kelvin source** | —          | **TO-247-4L ✓**                | —        | p.1             |
| Drain-source voltage        | V_DS       | 650                            | V        | Max Ratings     |
| Gate-source range (op.)     | V_GS       | −5…0 / +18                     | V        | Rec. Op.        |
| Gate threshold              | V_GS(th)   | 2.0 / 2.7 / 4.0                | V        | Elec.           |
| **R_ds(on) @ 25 °C**        | R_DS(on)   | **32**                         | mΩ       | Elec. (VGS=18V) |
| **R_ds(on) @ 175 °C**       | R_DS(on)   | **49**                         | mΩ       | Elec. (VGS=18V) |
| Internal gate resistance    | R_G        | 5.0                            | Ω        | Elec.           |
| Output capacitance          | C_oss      | 114                            | pF       | Elec.           |
| Total gate charge           | Q_g        | 55                             | nC       | Elec.           |
| **Turn-on energy (given)**  | E_on       | **31**                         | µJ       | Elec. (175 °C)  |
| **Turn-off energy (given)** | E_off      | **25**                         | µJ       | Elec. (175 °C)  |
| E-test conditions           | —          | 400 V, 15 A, 175 °C, R_g=4.7 Ω | —        | Elec.           |
| Rise / fall time            | t_r / t_f  | 12 / 9                         | ns       | Elec.           |
| **Body-diode fwd drop**     | V_SD       | **4.5**                        | V        | Diode (25 °C)   |
| **Reverse recovery charge** | Q_rr       | 72                             | nC       | Diode           |

_Notes — E_on/E_off are given directly. The M3S is a planar SiC device recommended for 18 V gate drive (works at 15 V); R_ds(on) here is specified at V_GS = 18 V, so the drive scheme must match to realise this resistance. This study uses onsemi's 175 °C E-values (E_on 31 µJ, E_off 25 µJ), matching the hot condition of the Wolfspeed data and the final comparison; the 25 °C typical set (E_on 33 µJ, E_off 16 µJ) is noted for reference only._

# **2 · Reasoning & calculations**

Two loss families are computed at the application point (200 V / 10 A / f_sw): conduction and switching.

## **2.1 · Conduction loss**

Conduction loss uses the RMS current and the hot resistance. For a hard-switched pulse, I_rms = I_peak·√D with D = 0.5 applied identically across candidates.

_I_rms = 10 · √0.5 = 7.07 A_

_P_cond = R_ds(on)@175°C × I_rms² = 0.049 × (7.07)² = 0.049 × 50 = 2.45 W_

_This is the lowest conduction loss of the three candidates — a direct consequence of the 32 mΩ die (vs 45–48 mΩ). Note it assumes the 18 V drive that the R_ds(on) spec requires._

## **2.2 · Switching loss**

E_on and E_off are given directly. Rescale from the datasheet test point to the application point; E ∝ V·I.

**Datasheet test point: 400 V, 15 A. Application point: 200 V, 10 A.**

_k = (200/400) × (10/15) = 0.5 × 0.667 = 0.333_

_E_on = 31 × 0.333 = 10.3 µJ E_off = 25 × 0.333 = 8.3 µJ_

_E_on + E_off ≈ 18.6 µJ per cycle_

_P_sw = (E_on + E_off) × f_sw_

_Caveat: E-values are now onsemi's 175 °C set, so the switching estimate sits on the same hot footing as the Wolfspeed data and the final comparison. The 25 °C typical figures (E_on + E_off ≈ 16.3 µJ → P_sw 1.63 W at 100 kHz) understate loss by ~13 % and are kept only as a reference floor. Both remain first-order until replaced by measured DPT energies._

## **2.3 · Which frequency**

P_cond is frequency-independent; P_sw grows linearly with f_sw. Report both; design against the 100 kHz worst case.

# **3 · Loss summary (values to compare)**

_Operating assumptions: 200 V, I_peak = 10 A, D = 0.5 (I_rms = 7.07 A), R_ds(on)@175 °C = 49 mΩ. E-values are onsemi's 175 °C set._

| **Quantity**                 | **@ 50 kHz** | **@ 100 kHz** | **Unit** |
| ---------------------------- | ------------ | ------------- | -------- |
| E_on (rescaled, 200 V/10 A)  | 10.3         | 10.3          | µJ       |
| E_off (rescaled, 200 V/10 A) | 8.3          | 8.3           | µJ       |
| E_on + E_off                 | 18.6         | 18.6          | µJ       |
| I_rms (D = 0.5)              | 7.07         | 7.07          | A        |
| **P_sw (switching loss)**    | **0.93**     | **1.86**      | W        |
| **P_cond (conduction loss)** | **2.45**     | **2.45**      | W        |
| **P_tot = P_cond + P_sw**    | **3.38**     | **4.31**      | W        |

**Secondary comparison axes (for tie-breaks):**

| **Axis**       | **Value** | **Comment**                                   |
| -------------- | --------- | --------------------------------------------- |
| Body diode V_F | 4.5 V     | Dead-time conduction loss                     |
| Q_rr           | 72 nC     | Lowest recovery charge of the three           |
| C_oss          | 114 pF    | Low                                           |
| Q_g            | 55 nC     | Moderate; needs 18 V drive for rated R_ds(on) |