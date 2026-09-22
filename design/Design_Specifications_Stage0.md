# Design Specifications - Stage 0

## 1. Operating point
- DC link voltage (nominal / max test): 200 V (architecture 800 V-ready)
- Target switching current: 10 A
- Switching frequency (if applicable): 50 kHz - 100 kHz
- Test type: Double Pulse Test (only the low-side device is actively switched; the high-side body diode freewheels the inductor current)

## 2. SiC MOSFET selection
Requirement: TO-247-4 with Kelvin source pin (mandatory).
Candidates:
- Wolfspeed C3M0045065K
- Infineon IMZA65R048M1H
- onsemi NTH4LN032N065M3S

Decision: **onsemi NTH4LN032N065M3S**
- Key ratings: 650 V, 32 mΩ, TO-247-4L (thin leads, Kelvin source), recommended gate drive +18 V.
- Datasheet parameters used downstream: C_iss = 1410 pF, Q_GD = 14 nC, Q_G = 55 nC, internal R_G = 5.0 Ω.
- Pinout (CASE 340CW): 1 = D, 2 = S2 (power source), 3 = S1 (Kelvin source), 4 = G.

## 3. Gate driver
Candidates:
- TI UCC21520
- TI UCC21750

Decision: **TI UCC21750** — single-channel isolated driver (±10 A, split outputs OUTH/OUTL, active Miller clamp, DESAT). One driver per switch.
- Gate resistor (Rg on / off): ~5 Ω each — provisional, **socketed** for the DPT sweep (sweep points 2.2 / 5 / 10 / 15 Ω).
- Drive voltage (+V / -V): +18 / -4
- Peak gate-current check: I_peak = V_drive / R_g,total = 22 V / 10.7 Ω ≈ 2.1 A (well under the ±10 A driver limit; internal terms already ≈ 5.7 Ω, so the current limit is never binding).

## 4. Gate loop design
- Loop area minimization strategy: minimize the commutation loop inductance — place the HF DC-link decoupling capacitor directly at the switch node, keep the gate loops short and tight, and locate the OUTH/OUTL resistors close to the gate. (Loop inductance, not the load inductance, drives turn-off overshoot.)
- Kelvin source connection: driver COM tied to the MOSFET **Kelvin-source pin (S1)**, kept separate from the power-source pin (S2) and the power-return path. This split is the sole reason for the 4-pin package.

## 5. DC link
- Capacitor type / value: **film (metallized polypropylene / MKP)**, ~100 µF bulk realized as a **parallel bank** (5 × 22 µF / 630 V, e.g. TDK B32776P6226K000), rated ≥ 500 V (2× derating of the 200 V bus).
  - Sizing: Q = ½·I·t = ½ × 10 A × 25 µs = 125 µC; C ≥ Q/ΔV = 125 µC / 2 V ≈ 63 µF → rounded to ≈ 100 µF (~1.25 V sag).
- Decoupling strategy: **two-population** — bulk film for energy storage + small HF film/ceramic at the switch node for the fast transient. Driver rails decoupled with ceramic MLCCs: 2 × 15 µF (or 22 µF) / 35 V on VDD–COM and VEE–COM, ≥ 1 µF / ≥ 25 V on VCC–GND. Voltage of each decoupling cap set by 2× derating of its *local* node; MLCC DC-bias derating accounted for (values chosen to stay > 10 µF under bias).

## 6. Inductive load
- Inductance value: **500 µH** — from L = V·Δt/ΔI = 200 V × 25 µs / 10 A (di/dt = 0.4 A/µs, ~25 µs ramp to 10 A).
- Construction / sourcing: single-winding power inductor with **I_sat ≥ 15 A**, connected **off-board via a 2-pin terminal**.
  - Primary candidate: toroid 500 µH / 15 A (Bel SAFTC-15-0501, single-winding confirmed on datasheet) — subject to stock/lead time.
  - Fallback: hand-wound single-layer **air-core coil** (cannot saturate; geometry from Wheeler's formula, value verified with an LCR meter).

## 7. Simulation
- LTspice model status: to be built next — double-pulse test from the completed schematic (two-pulse VPulse on the driver input; device model + relevant parasitics).
- Key results: TBD — E_on / E_off and V_DS overshoot vs R_g (sweep 2.2 / 5 / 10 / 15 Ω), compared against the hand-calculated switching times (~19 ns at 5 Ω).

---
**Scope note:** physical PCB fabrication deferred (cost / time). Stage 0 is validated by design calculations and LTspice simulation; the schematic, custom symbol/footprint, and full BOM are complete.
