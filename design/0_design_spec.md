# Design Specifications - Stage 0

## 1. Operating point
- DC link voltage (nominal / max test): 200 V
- Target switching current: 10 A
- Switching frequency (if applicable): 50 kHz- 100 kHz
- Test type: Double Pulse Test

## 2.SiC MOSFET selection
Requirement: T0-247-4 with Kelvin source pin (mandatory).
Candidates:
- Wolfspeed C3M0045065K
- Infineon IMZA65R048M1H
- onsemi NTH4LN032N065M3S

Decision : onsemi NTH4LN032N065M3S

## 3.Gate driver
- Candidates:
- TI UCC21520
- UCC21750
- Gate resistor (Rg on/off): 
- Drive voltage (+V / -V): +18 / -4

## 4. Gate loop design 
- Loop area minimization strategy: TBD
- Kelvin source connection: TBD

## 5. DC link
- Capacitor type / value: TBD
- Decoupling strategy : TBD

## 6. Inductive load
- Inductance value: TBD
- Construction / sourcing: TBD

## 7.Simulation
- LTspice model status: TBD
- Key results: TBD
