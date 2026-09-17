# Gate driver compatibility — UCC21520 & UCC21750 vs onsemi NTH4LN032N065M3S

*SiC half-bridge inverter leg — DPT characterization · Stage 0 (design study)*

Both candidate gate drivers are checked against the selected switch
(onsemi NTH4LN032N065M3S, 650 V / 32 mΩ SiC, TO-247-4L). Compatibility is
evaluated on seven axes: can the driver deliver the required gate swing, fast
enough, without exceeding any limit, and survive the electrical environment the
SiC creates.

## UCC21520 / onsemi NTH4LN032N065M3S

| Axis | What I compare | UCC21520 | onsemi MOSFET | Verdict |
|---|---|---|---|---|
| **Gate voltage rails** — can it deliver +18/−4 without exceeding V_GS max? | Driver VDD−VSS rail vs MOSFET recommended & abs-max V_GS | VDD−VSS < 25 V rec (30 V abs) | Rec. −5…0/+18 V; abs max −10/+22.6 V static | ✅ +18/−4 → 22 V rail (< 25 V), gate well inside −10/+22.6 |
| **UVLO matches the drive scheme** (the SiC-critical one) | Driver UVLO vs VGS below which R_DS(on) balloons | 8 V version: 8.5 V rise / 7.9 V fall | 32 mΩ@18 V but **41 mΩ@15 V** — very VGS-sensitive | ⚠️ Use the **8 V UVLO part (UCC21520), not the 5 V "A" variant** — see note |
| **Peak current / drive strength** | Peak I_G = ΔV_GS / R_g,total vs driver peak | 4 A source / 6 A sink; R_OH = 5 Ω, R_OL = 0.55 Ω | R_g,int = 5 Ω | ✅ Even at R_ext = 0: turn-on ≈ 22/10 = 2.2 A, turn-off ≈ 22/5.5 ≈ 4 A — under limits; driver never the bottleneck |
| **Average drive power** | P = Q_g·ΔV_GS·f_sw vs driver P_D and bias-supply current | 450 mW/side; ~950 mW total | Q_g = 55 nC | ✅ 55n·22·100k ≈ **0.12 W/switch**; I_avg ≈ 5.5 mA/ch — trivial |
| **CMTI vs dV/dt** | Switching-node dV/dt vs driver CMTI | **> 125 V/ns** | ~10–30 V/ns expected at 200 V bus | ✅ Comfortable, provided R_g keeps dV/dt under 125 V/ns |
| **Timing: dead-time, prop delay, min pulse** | DPT / dead-time plan vs driver timing | Prog. DT = 10·R_DT(kΩ); t_pd 33 ns; PWD 6 ns; t_pw,min 20 ns | Dead-time target 100–200 ns | ✅ 150 ns → R_DT = 15 kΩ; delays negligible vs µs DPT pulses |
| **False turn-on protection** | MOSFET V_GS(th) & Miller vs driver clamp features | **No integrated Miller clamp** | V_GS(th) = **2.0 V min** (low!) | ⚠️ Compatible **only with negative off-bias** — see note |

### The two judgement calls

- **UVLO subtlety (row 2).** SiC R_DS(on) collapses only near full drive, so the
  device must never operate far below 18 V. The UCC21520's UVLO watches
  VDD−**VSS**, so with a −4 V floor the 8.5 V trip really means the positive rail
  can sag to ~4.5 V before lockout — not ideal in theory. In practice, with a
  well-regulated isolated bias supply both rails move together and it is a
  non-issue for a ≤200 V bench DPT. Honest note: a production traction inverter
  would often use a driver with a dedicated ~13 V SiC UVLO (e.g. the UCC21750,
  evaluated below).
- **Miller clamp (row 7).** The UCC21520 has no active Miller clamp, so with this
  onsemi part's low 2.0 V threshold the only defence against dV/dt-induced false
  turn-on is the −4 V negative bias plus a tight, low-inductance gate loop.
  Workable, but it leaves no on-chip safety net — the main reason to also look at
  a driver that has one (below).

## UCC21750 / onsemi NTH4LN032N065M3S

Same compatibility check, now against the traction-grade UCC21750. It is a
single-channel isolated SiC/IGBT driver, so a half-bridge leg needs two of
them — but it adds protection and sensing the UCC21520 does not have.

| Axis | What I compare | UCC21750 | onsemi MOSFET | Verdict |
|---|---|---|---|---|
| **Gate voltage rails** — can it deliver +18/−4 without exceeding V_GS max? | Driver VDD−VEE rail vs MOSFET recommended & abs-max V_GS | VDD−VEE ≤ **33 V** rec (36 V abs); split output (VDD + VEE rails) | Rec. −5…0/+18 V; abs max −10/+22.6 V static | ✅ +18/−4 → 22 V rail (well under 33 V), large headroom; gate well inside −10/+22.6 |
| **UVLO matches the drive scheme** (the SiC-critical one) | Driver UVLO vs VGS below which R_DS(on) balloons | **12 V** VDD UVLO (12.0 V rise / 10.7 V fall), referenced to VDD−**COM** (the source) | 32 mΩ@18 V but **41 mΩ@15 V** — very VGS-sensitive | ✅ Source-referenced 12 V UVLO → gate never runs far below 15 V. SiC-correct; **fixes the UCC21520 caveat** |
| **Peak current / drive strength** | Peak I_G = ΔV_GS / R_g,total vs driver peak | **±10 A** source/sink; split OUTH/OUTL; R_OH,eff ≈ 0.7 Ω, R_OL ≈ 0.3 Ω | R_g,int = 5 Ω | ✅ 10 A far exceeds needs; pick R_ext for dV/dt; driver never the bottleneck (more headroom than UCC21520) |
| **Average drive power** | P = Q_g·ΔV_GS·f_sw vs driver P_D and bias-supply current | ~965 mW/side; ~985 mW total | Q_g = 55 nC | ✅ 55n·22·100k ≈ **0.12 W/switch**; I_avg ≈ 5.5 mA/ch — trivial |
| **CMTI vs dV/dt** | Switching-node dV/dt vs driver CMTI | **> 150 V/ns** | ~10–30 V/ns expected at 200 V bus | ✅ Even more margin than UCC21520; keep R_g sensible |
| **Timing: dead-time, prop delay, min pulse** | DPT / dead-time plan vs driver timing | t_pd 90 ns; **part-to-part skew 30 ns**; no DT pin (dead-time set in controller / IN+/IN− interlock); 40 ns input deglitch | Dead-time target 100–200 ns | ⚠️ No DT pin → set dead-time in controller; 30 ns skew eats budget (fine for µs DPT pulses) |
| **False turn-on protection** | MOSFET V_GS(th) & Miller vs driver clamp features | **Internal active Miller clamp** (CLMPI, trips 2 V above VEE) + active pulldown | V_GS(th) = **2.0 V min** (low!) | ✅ On-chip clamp removes the false-turn-on worry (**fixes the UCC21520 gap**); negative bias still used |

### What changes vs the UCC21520

- **UVLO now referenced to the source (row 2).** The 12 V UVLO is measured
  VDD−COM, i.e. against the Kelvin source, so it truly guarantees the gate is
  never driven far below 15 V — the region where this onsemi part's R_DS(on)
  climbs (32 mΩ → 41 mΩ). This removes the marginal-UVLO caveat we had with the
  UCC21520.
- **Internal Miller clamp (row 7).** With V_GS(th) as low as 2.0 V, the on-chip
  active clamp (CLMPI, trips 2 V above VEE) adds a low-impedance path against
  dV/dt-induced false turn-on — we no longer rely on negative bias alone.
- **Bonus protection & sensing (not in the table).** DESAT short-circuit
  detection with 400 mA soft turn-off, FLT fault reporting, and an isolated
  AIN→APWM channel that can read a thermal diode / NTC across the barrier — the
  last of which lines up with the Stage-2 junction-temperature (Tj) estimator.
- **Cost: added complexity.** Single-channel → two ICs per leg; DESAT needs a
  blanking network (HV diode, C_blk, series R, clamp) when enabled; 30 ns
  part-to-part skew. On a first board that is more to get right — but DESAT can
  be tied to COM (disabled) for first bring-up and added once the board is
  proven.

### Decision

I will use the **UCC21750**. It responds better to the specific demands of the
onsemi NTH4LN032N065M3S: the source-referenced 12 V UVLO and the internal Miller
clamp directly resolve the two caveats the UCC21520 left open (marginal UVLO
under negative bias, and no clamp against the low 2.0 V threshold). It will make
the design more complicated — a single-channel part means two drivers per leg,
and the DESAT and sensing features add parts and layout work — but for a board
whose purpose is to demonstrate traction-inverter readiness, that is a good
trade-off. The extra protection and the isolated Tj sensing also line up with the
Stage-2 junction-temperature estimator.
