# TL082 Controlled Half-Bridge Charge-Pump Voltage Doubler (12V to 24V DC-DC Converter)

A **12V to 24V DC-DC voltage doubler** built around a **switched-capacitor (charge-pump)** topology, a **TL082 JFET op-amp** oscillator, **2N2222 transistor gate drivers**, and an **IRFZ44N N-channel MOSFET half-bridge** with a **bootstrap high-side drive**. This open hardware project is designed as a stable, efficient and low-cost boost converter for hobbyists, electronics students and embedded-systems makers who need a simple, inductor-free way to double a 12V supply.

The oscillator, driver stage and power stage are arranged so that the two MOSFETs are triggered in **complementary (anti-phase)** fashion: while one conducts, the other is off.

> **Keywords:** charge pump voltage doubler, 12V to 24V converter, switched-capacitor DC-DC, TL082 oscillator, TL082 square wave generator, IRFZ44N half-bridge, MOSFET bootstrap gate drive, inductorless boost converter, op-amp astable multivibrator, DIY DC-DC boost converter, LTSpice charge pump.

> **Important note:** On a solderless breadboard this design may be unstable because of parasitic capacitance, high contact resistance and current limits (about **1A per channel**). For full performance and stability, simulate it in **LTSpice** first and then build it **hard-wired (point-to-point soldered) on a 7x10 cm perfboard**.

---

## Table of Contents

1. [Features](#features)
2. [How a Charge-Pump Voltage Doubler Works](#how-a-charge-pump-voltage-doubler-works)
3. [Circuit Architecture and Component Placement](#circuit-architecture-and-component-placement)
4. [Bill of Materials](#bill-of-materials)
5. [Stability and Build Recommendations](#stability-and-build-recommendations)
6. [Troubleshooting](#troubleshooting)
7. [Limitations](#limitations)
8. [Possible Applications](#possible-applications)
9. [FAQ](#faq)
10. [Roadmap](#roadmap)

---

## Features

- Doubles a **12V DC input to about 24V DC** (ideal; real output is lower by the diode drops and load).
- **Inductor-free**: only capacitors, diodes and MOSFETs in the power path.
- **Self-oscillating**: no microcontroller or dedicated PWM IC required.
- **Complementary drive** from a dual JFET op-amp (TL082): one section generates the square wave, the other inverts it.
- **Bootstrap high-side gate drive** so the upper MOSFET gets enough V<sub>GS</sub> (about 10V).
- Built from cheap, widely available parts (TL082, 2N2222, IRFZ44N, 1N4148-class diodes).
- Suitable for learning about switched-capacitor converters, op-amp oscillators and half-bridge layout.

---

## How a Charge-Pump Voltage Doubler Works

A charge pump moves energy with capacitors instead of an inductor:

1. **Charge phase:** The lower MOSFET conducts and pulls the switching node to GND. The flying capacitor's (-) terminal is at GND, so it charges from +12V through the input diode to roughly 12V (minus the diode drop).
2. **Pump phase:** The upper MOSFET conducts and pulls the switching node up to +12V. The flying capacitor's (-) terminal is now at 12V, so its (+) terminal rises to about 24V.
3. **Output:** That raised voltage passes through the output diode and charges the output capacitor, giving roughly **24V DC** minus diode drops and load-dependent droop.

Because the flying capacitor is stacked on top of the input voltage every cycle, the output approaches **2 x V<sub>in</sub>** at light load.

---

## Circuit Architecture and Component Placement

### 1. Oscillator and Phase Inverter Stage (TL082 JFET Op-Amp)

**Section A - square wave generator (astable oscillator)**
- A **15kΩ resistor** between Pin 1 (output) and Pin 2 (inverting input), together with a **104 (100nF) capacitor** from Pin 2 to GND, sets the main oscillation frequency.
- A **150kΩ resistor** between Pin 1 and Pin 3 (non-inverting input) provides **positive-feedback hysteresis**.
- For reliable start-up oscillation (common-mode bias), Pin 3 is biased to a soft **6V reference** by two **10kΩ resistors** from +12V and GND.
- *Rough theoretical estimate:* with these values the hysteresis ratio is small and the frequency lands in the **low-kHz range (on the order of 5 kHz)**. Treat this as a starting point; verify with simulation and an oscilloscope, since the TL082 is not rail-to-rail and real thresholds differ.

**Section B - inverter stage**
- Designed to run in exact **anti-phase** with Section A (one MOSFET turns on while the other turns off).
- The TL082 is **not rail-to-rail**, so its upper and lower output limits must be handled carefully to avoid getting stuck in the linear region. For this reason Pin 5 (non-inverting input) is held at a soft **3.24V reference** using a **27kΩ** resistor from +12V and a **10kΩ** resistor to GND.
- A **180kΩ high-gain resistor** between Pin 6 and Pin 7 (output) forces the op-amp into full saturation, producing a sharp, inverted square wave.

### 2. Power MOSFET Driver Stage (2N2222)

- Current from the op-amp outputs (Pin 1 and Pin 7) goes through **10kΩ base resistors** into the **2N2222** transistors.
- The **lower transistor's collector** is tied to +12V through a **9.1kΩ pull-up** and drives the **low-side MOSFET**.
- The **upper transistor's collector** is fed from a **bootstrap supply** so that the **high-side MOSFET** can reach the minimum **~10V gate-source voltage (V<sub>GS</sub>)**. Using a fast diode (for example **1N4148**) and a small **1uF-10uF** capacitor, the bootstrap rail lifts the upper MOSFET's gate to about **22V-24V**.

### 3. Power and Boost Stage (IRFZ44N)

- **Half-bridge layout:** The **drain of MOSFET 1** connects directly to the main **+12V** rail. The **source of MOSFET 1** and the **drain of MOSFET 2** are joined with a thick trace to form the **switching node (midpoint)**. The **source of MOSFET 2** goes straight to **GND**.
- **Voltage pumping:** The **(-) terminal of the main 10uF flying capacitor** connects to the switching node. Its **(+) terminal** is kept off the transistor pins entirely and floats at the **pump node**, where the two power diodes meet. Current flows from +12V through the **input diode** (its cathode to the capacitor's positive side), and the boosted voltage leaves through the **output diode** into a **100uF output capacitor**, giving a stable **~24V DC** output.

---

## Bill of Materials

| Qty | Part | Value / Type | Role |
|----|------|--------------|------|
| 1 | TL082 | Dual JFET op-amp | Oscillator and inverter |
| 2 | 2N2222 | NPN transistor | MOSFET gate drivers |
| 2 | IRFZ44N | N-channel power MOSFET | Half-bridge switches |
| 2 | Power diode | Fast / Schottky recommended | Input and output pump diodes |
| 1 | UF4007 | Fast signal diode | Bootstrap diode |
| 1 | Capacitor | 10uF | Flying capacitor |
| 1 | Capacitor | 100uF | Output capacitor |
| 1 | Capacitor | 1uF-10uF | Bootstrap capacitor |
| 1 | Capacitor | 104 (100nF) | Oscillator timing |
| 1 | Resistor | 15kΩ | Oscillator timing |
| 1 | Resistor | 150kΩ | Hysteresis (positive feedback) |
| 1 | Resistor | 180kΩ | Inverter gain |
| 1 | Resistor | 27kΩ | 3.24V reference (upper) |
| 5 | Resistor | 10kΩ | 6V bias divider (x2), 3.24V reference (lower), base resistors (x2) |
| 2 | Resistor | 9.1kΩ | Low-side pull-up |

Quantities are taken from the schematic description; adjust to your final layout. Use a **Schottky diode** for the pump diodes if you want a higher output voltage and better efficiency.

---

## Stability and Build Recommendations

1. **Suppress parasitics:** Keep signal lines and power lines physically separated on the perfboard. Reinforce high-current MOSFET paths with thick solder bridges or heavy wire.
2. **Simulate first:** Validate op-amp reference levels and switching behavior in **LTSpice** before building. Simulation outputs for this schematic will be added to this repository soon.
3. **Decouple the supply:** Place a ceramic capacitor (100nF) close to the TL082 supply pins and a bulk capacitor on the +12V input.
4. **Keep the switching loop small:** The flying capacitor, MOSFETs and diodes carry the pulsed current, so keep that loop short.
5. **Heatsinking:** At higher loads the IRFZ44N and diodes warm up; add a small heatsink if needed.

---

## Troubleshooting

| Symptom | Likely cause | What to check |
|---------|--------------|---------------|
| No oscillation | Bias network wrong | Pin 3 should sit near 6V; check the two 10kΩ resistors |
| Both MOSFETs on or both off | Inverter stuck in linear region | Pin 5 should be near 3.24V; check the 180kΩ feedback resistor |
| Output stuck near 12V | Upper MOSFET not fully on | Check bootstrap diode and capacitor, and gate voltage on the high-side MOSFET |
| Output much below 24V | Diode drops or heavy load | Use Schottky diodes; reduce load; use a larger flying capacitor |
| Unstable on breadboard | Parasitics, poor contacts | Move to soldered perfboard |

---

## Limitations

- Maximum current is limited to about **1A per channel** in this design.
- Output voltage **sags under load** and is not regulated; add a regulator or feedback loop if you need a precise 24V.
- Output ripple depends on switching frequency, flying capacitance and load; measure it on your build.
- Breadboard prototypes are unreliable at higher currents.

---

## Possible Applications

- Powering 24V relays, small fans or sensors from a 12V automotive or battery source.
- Bias or gate-drive supply rails for other circuits.
- Teaching projects on switched-capacitor converters, op-amp oscillators and bootstrap drivers.
- Low-power auxiliary rail in embedded and IoT hardware.

---

## FAQ

**What does this circuit do?**
It converts a 12V DC input into roughly 24V DC using a charge pump (voltage doubler) driven by a TL082 oscillator and an IRFZ44N half-bridge.

**Why use a bootstrap circuit?**
The high-side N-channel MOSFET's source sits at the switching node, which rises to 12V. Its gate must therefore go well above 12V (about 22V-24V here) to reach the needed V<sub>GS</sub>.

**Why is the TL082 not rail-to-rail a problem?**
Its output cannot swing fully to the supply rails, so reference voltages (6V and 3.24V) are chosen to keep both sections reliably switching instead of settling in the linear region.

**Can I build it on a breadboard?**
For testing at very low current, yes, but expect instability. A soldered 7x10 cm perfboard is strongly recommended.

**Will it give exactly 24V?**
No. The ideal value is 2 x V<sub>in</sub>, but diode drops and load reduce it.

---

## Roadmap

- [ ] Add LTSpice schematic and simulation results
- [ ] Add full schematic image and perfboard layout
- [ ] Add measured output voltage, ripple and efficiency
- [ ] Test Schottky diodes and larger flying capacitors

---

## Contributing

Issues, suggestions and pull requests are welcome. If you build this circuit, please share your measurements and layout photos.

## ⚖️ Governance & License
This ecosystem is maintained under the **MIT License** and managed by **Zaritech Labs**. 

*May Allah grant us success on this path.*
