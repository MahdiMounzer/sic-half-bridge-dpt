# SiC Half-Bridge Inverter Leg + Double Pulse TEST (DPT)

A documented design-to-characterization project: a SiC half-bridge inverter leg with low-inductance 4-layer PCB, progressive bring-up, and DPT characterization (Eon/Eoff, dv/dt, overshoot, conduction losses).

## Motivation
Portfolio piece targeting EV / power electronics roles. Every design decision, measurement, and failure is documented as it happens.

## Status
**Stage0 - Design study** (operation point, device selection, gate loop, DC link, inductive load, LTspice simulation).

## Roadmap
1. Design study (Stage 0)
2. Schematic capture (KiCad)
3. 4-layer PCB - low loop inductance, kelvin source, separated power/signal
4. Progressive bring-up (20V -> 50V -> 100V -> 200V)
5. DPT characterization

## Structure
- 'design/' - design study and specifications
- 'schematic/' - KiCad schematic
- 'pcb/' - KiCad PCB layout 
- 'simulation/' - LTspice files
- 'measurements/' - DPT data, scope captures 
- 'logbook/' - chronological notes, failures learnings 

## Key references
- Wolfspeed CPWR-AN25
- TI TIDA-01605
