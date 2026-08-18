# SiC Half-Bridge Double Pulse Test — Project Overview

*The plain-language introduction to the design study. Read this before the detailed constraint blocks. It explains what is being built, why, how it works, and which decisions can be varied to observe their effect.*

---

## 1. What am I building?

One switching cell.

Two transistors stacked between the plus and minus of a DC supply, with the middle point between them going out to a load. That two-transistor stack is called a **half-bridge leg**, and it is the single most important building block in power electronics. A complete EV traction inverter is simply **three of these legs side by side** — one per motor phase. Build and fully understand one leg, and the three-phase inverter is just that leg repeated.

So this project is not a "smaller inverter." It is *the fundamental unit the inverter is made of*, built as a laboratory instrument so its behaviour can be measured precisely.

## 2. Why build it — what am I trying to learn?

When a transistor switches a real load on and off, the interesting and destructive physics happens in roughly **20 nanoseconds** — the transition itself, not the steady on-state or off-state. During that transition the device has significant voltage across it **and** significant current through it at the same time, so it dissipates energy, rings, overshoots, and heats. Two things live in that tiny window:

- **Switching loss** — energy burned every time the device turns on or off. In a real inverter this happens tens of thousands of times per second, so it dominates efficiency.
- **The SiC advantage** — silicon-carbide devices switch far faster and with far lower loss than the older silicon IGBTs traction inverters historically used. That advantage *is* the 20 ns transition. Measuring it is the point.

The problem: in a running inverter you cannot cleanly observe those 20 ns — too many things happen at once, continuously. So power electronics uses a controlled laboratory technique to isolate a single switching event.

## 3. How it works — the Double Pulse Test

The technique is the **Double Pulse Test (DPT)**. Instead of running the transistor continuously, you fire it exactly **twice**, in a deliberately arranged circuit, so an oscilloscope can capture one clean switching transition.

The sequence:

- **Pulse 1 (long):** the device under test is switched on. With an inductor as the load, the current ramps up linearly — like slowly filling a tank. When the current reaches the target value (e.g. 10 A), the pulse ends. Pulse 1 exists only to *establish a known current*.
- **The gap (device off):** the current cannot stop instantly — an inductor forces it to keep flowing. It "freewheels" through the other transistor's built-in diode, staying at roughly 10 A. This is the moment the oscilloscope captures the **turn-off** event.
- **Pulse 2 (short):** the device is switched on again, now against a known 10 A already flowing. This captures the **turn-on** event, including how the freewheeling diode recovers.

Two pulses, and you have both the turn-on and turn-off transitions, at a known voltage and known current, frozen on the scope. From those waveforms come the switching-energy numbers (E_on, E_off), the voltage overshoot, the switching speed (dV/dt), and the diode's recovery behaviour.

**The one sentence to remember: everything in this project exists to catch a single switching event on the oscilloscope, cleanly and safely. That measurement is the whole deliverable.**

## 4. What each part is for

Every component and every number is either *setting the conditions of the measurement* or *making the measurement possible and safe*. Grouped by role:

**Setting up the shot — the conditions you switch under:**

- **Bus voltage** — how hard you push the device. Higher voltage gives more realistic, more dramatic data, but more stress on everything and a more dangerous failure (fault energy grows with the square of voltage).
- **Test current** — how much current you interrupt. More current means a bigger, clearer signal to measure, but more stored energy and a bigger event if something fails.

**Making the shot possible — the supporting cast:**

- **Load inductor** — the trick that forces a known current to flow at the instant of switching. It behaves as a "current source you build out of time": hold the device on for a set duration, and the current reaches a set value.
- **DC-link capacitor** — a local energy reservoir sitting right next to the transistors, so the bus voltage does not sag during the fast pulse. Without it, "200 V" would not actually be 200 V at the moment that matters.

**Taking the shot — the actors:**

- **Gate driver** — the circuit that physically flips the transistor on and off, fast and cleanly. A transistor gate needs a surprisingly large, brief burst of current to switch quickly; the driver delivers it, and shapes how fast the switching happens.
- **SiC MOSFET** — the **subject of the entire experiment**. This is the device being characterized. Everything else is scaffolding built around it to get the cleanest, safest picture of it switching.

## 5. What to vary — and what each variation reveals

This is where the project becomes a *study* rather than a single measurement. Once the board works, the value comes from sweeping variables and watching the switching event change. The main axes:

**Bus voltage (e.g. 100 V → 200 V → 400 V).**
Switching losses scale with voltage. Sweeping the bus produces a *curve* of E_on and E_off versus voltage — the single most useful result for predicting behaviour at the real automotive operating point. It also shows the voltage overshoot growing, which is the safety limit that governs how high you can go.

**Test current (e.g. 5 A → 10 A → 20 A).**
Switching losses also scale with current. Sweeping current gives E_on and E_off versus load — together with the voltage sweep, this fully maps the device's switching-loss surface. It also shows how the diode recovery worsens with current.

**Gate resistance (R_g).**
This is the knob that controls switching *speed*. A small gate resistor switches fast — low loss, but violent overshoot and ringing and more electromagnetic noise. A large resistor switches gently — clean and quiet, but higher loss. Sweeping R_g reveals the fundamental **loss-versus-overshoot trade-off**, and produces one of the most instructive graphs in the whole project: there is no free lunch, only a chosen operating point on this curve.

**The device itself (comparing MOSFET candidates).**
Swapping the transistor under test — different R_ds(on), different internal capacitances, different gate charge — shows directly how device choice changes conduction loss, switching loss, and overshoot. This is the comparison that ultimately justifies picking one device for a real design, backed by *your own measurements* rather than datasheet claims.

**Dead time (the gap between the two transistors being on).**
Too short risks both transistors conducting at once (a destructive short across the bus); too long wastes efficiency and distorts the output. Varying it shows the safe operating window.

**Secondary factors worth observing rather than sweeping:**

- **Load inductor construction** (air-core vs ferrite) — affects how clean the switch-node waveform is, because the inductor's own parasitics ring against the circuit.
- **Decoupling capacitor placement** — how physically close the small high-speed capacitors sit to the transistors sets the loop inductance, which directly sets the overshoot.

## 6. The through-line

Strip away the equations and the project is one loop:

*Build the fundamental switching cell of an inverter → use the double pulse test to freeze one switching event → measure the loss, speed, and overshoot in that event → then vary voltage, current, gate speed, and device, and watch how the event changes.*

Every parameter discussed in the detailed constraint blocks is either a condition of that measurement or a variable to sweep in it. Keeping that in view is what turns a page of numbers back into a single, understandable experiment.
