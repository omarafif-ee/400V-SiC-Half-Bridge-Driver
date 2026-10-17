# 400 V SiC MOSFET Isolated Half-Bridge Gate Driver PCB

![PCB 3D Render](Media/PCB_3D_Render.png)

## Overview

This project covers the design and simulation of a 400 V isolated half-bridge gate driver using KiCad 10.0 and LTspice 26.1. A Texas Instruments `UCC21520DW` isolated dual-channel gate driver controls two Wolfspeed `C3M0120090D` 900 V SiC MOSFETs.

The design focuses on keeping the 5 V control circuit separate from the high-voltage power stage, providing a floating supply for the high-side gate, and setting the timing and gate resistances to reduce the risk of shoot-through. The PCB is a two-layer design. LTspice was used to examine the switching-node voltage, gate-drive waveforms, dead time, and Miller-related gate bounce.

📄 **[Read the Full Technical Report (PDF)](Documentation/Technical%20Report%20-%20400V%20SiC%20MOSFET%20Isolated%20Half-Bridge%20Gate%20Driver%20PCB.pdf)**

---

## Key Specifications

- **DC bus voltage:** 400 V DC
- **Power switches:** 2 × Wolfspeed `C3M0120090D` 900 V SiC MOSFETs in TO-247-3 packages
- **Gate driver:** Texas Instruments `UCC21520DW`, with a stated isolation rating of 5.7 kVrms
- **Gate-drive supply:** +15 V, with a bootstrap network for the floating high-side supply
- **Hardware dead time:** Approximately 220 ns, set using `R1` = 22 kΩ
- **PCB:** Two copper layers
- **High-voltage clearance rule:** Custom KiCad design rule set to 2.9 mm for `DC_PLUS` and `SW_NODE` against other nets, with an exception for connections within the same floating high-side section

---

## Hardware Design and PCB Layout

### 1. Schematic

![Schematic](Media/Schematic.png)

Since the circuit switches a 400 V bus, the control and power sides need to be kept separate. The `UCC21520DW` provides galvanic isolation between the logic ground (`GND`) and the power-side connections (`GNDPWR` and `SW_NODE`). The low-side MOSFET is referenced to ground, while the high-side driver floats with `SW_NODE`.

A bootstrap network provides the floating high-side supply without a separate power supply. When the low-side MOSFET turns on and `SW_NODE` falls to 0 V, the +15 V rail charges capacitors `C4` and `C5` through resistor `R8` and diode `D1`. When the high-side switch turns on, `D1` becomes reverse-biased and the charged capacitors supply the floating driver. With an estimated diode drop of 1 V, the high-side gate-drive voltage is about 14 V.

The gate resistors use separate paths for turn-on and turn-off. The 10 Ω resistors (`R2` and `R5`) control gate charging, while the Schottky diodes (`D2` and `D3`) and 1 Ω resistors (`R3` and `R6`) provide a lower-resistance turn-off path. The effective resistance of the 1 Ω and 10 Ω paths in parallel is about 0.91 Ω. This helps discharge the gate quickly when the MOSFET turns off. The 10 kΩ pull-down resistors (`R4` and `R7`) keep the gates near their source potentials when the driver is unpowered.

Resistor `R1` sets the driver's dead time. Using the approximate relationship of 10 ns/kΩ, a 22 kΩ resistor gives about 220 ns. Capacitor `C3` is placed in parallel with `R1` to help filter switching noise at the dead-time pin.

The 100 nF, 630 V capacitor `C8` is placed near the MOSFETs to absorb high-frequency switching transients. Two 220 kΩ resistors (`R9` and `R10`) are connected in series as bleeder resistors, so each sees approximately 200 V across the 400 V bus.

### 2. Two-Layer PCB Layout

![2D Layout](Media/PCB_2D_Layout.png)

The logic section is placed on the left side of the board and the power stage on the right. No traces are routed beneath the gate driver `U1`; this leaves more than 8 mm of separation between the logic-side and high-voltage connections.

A custom 2.9 mm minimum-clearance rule was applied to `DC_PLUS` and `SW_NODE` against other nets. Connections within the same floating 15 V high-side section, such as `VDDA`, `OUTA`, and `HS_GATE`, are treated as exceptions. The layout was checked in KiCad until no design-rule-check (DRC) errors remained.

The MOSFET pins have a centre-to-centre spacing of 5.45 mm, which limits the clearance available between their pads. To fit this area, the `DC_PLUS`, `SW_NODE`, and `GNDPWR` traces narrow to 1.5 mm near the MOSFET pads, then widen back to 2.0 mm after leaving the pads. `SW_NODE` is routed on the top copper layer between `C8` and the midpoint of `R9` and `R10`, without extra vias. These components and connector `J3` are kept near `Q1` and `Q2` to shorten the switching path and reduce parasitic inductance.

---

## LTspice Simulation

![LTspice Schematic](Media/LTspice_Schematic.png)

![LTspice Waveforms](Media/LTspice_Waveforms_Full.png)

The secondary-side half-bridge was simulated in LTspice over 20 µs, covering two switching periods at 100 kHz. To avoid convergence problems, the `UCC21520DW` and bootstrap circuit were replaced with floating pulse voltage sources. The high-side source was referenced to `SW_NODE` and pulsed between 0 V and 14 V to account for the estimated 1 V bootstrap diode drop. The low-side source was referenced to ground and pulsed between 15 V and 0 V. The pulses include 15 ns rise and fall times and maintain the 220 ns dead time.

The simulation uses a 400 V supply with 0.05 Ω series resistance, a 20 nH supply-loop inductance, and a 200 µH inductive load with 20 Ω series resistance. A custom LTspice VDMOS model was used for the `C3M0120090D` to represent its main switching characteristics and capacitances.

### Results

- **Switching-node voltage:** `SW_NODE` transitions between approximately 0 V and 400 V, while the DC bus remains close to 400 V.
- **High-side gate voltage:** The high-side gate reaches about 14 V above `SW_NODE`. When the switching node is at 400 V, the absolute gate voltage reaches approximately 414 V.
- **Dead time:** The drive waveforms maintain approximately 220 ns between switching events, and no shoot-through is seen in the simulation.
- **Low-side gate bounce:** The simulated Miller-related spike is around 0.7–0.8 V, below the 1.8 V minimum threshold cited in the report.

These results show the expected switching behaviour in the simulated circuit. The gate-drive IC and bootstrap network were simplified in the model, so physical testing would still be needed to verify the finished hardware.

---

## Bill of Materials (BOM)

| Ref. | Qty. | Value / Role | Footprint | Manufacturer | Part Number (MPN) |
|:---|:---:|:---|:---|:---|:---|
| **U1** | 1 | Isolated gate driver | SOIC-16W | Texas Instruments | `UCC21520DW` |
| **Q1, Q2** | 2 | 900 V SiC MOSFET | TO-247-3 | Wolfspeed | `C3M0120090D` |
| **D1** | 1 | Bootstrap diode | DO-41 axial | Generic | `UF4007` |
| **D2, D3** | 2 | Gate turn-off diode | SOD-123 SMD | Generic | `1N5819HW` |
| **C8** | 1 | 100 nF snubber capacitor, 630 V | Radial, 15 mm pitch | WIMA | `MKP1J031004B00KSSD` |
| **R9, R10** | 2 | 220 kΩ bleeder resistor | 0207 axial | Yageo | `MFR-25FTE52-220K` |
| **R1** | 1 | 22 kΩ dead-time resistor | 0805 SMD | Yageo | `RC0805FR-0722KL` |
| **R2, R5** | 2 | 10 Ω turn-on resistor | 0805 SMD | Yageo | `RC0805FR-0710RL` |
| **R3, R6** | 2 | 1 Ω turn-off resistor | 0805 SMD | Yageo | `RC0805FR-071RL` |
| **R4, R7** | 2 | 10 kΩ gate pull-down resistor | 0805 SMD | Yageo | `RC0805FR-0710KL` |
| **R8** | 1 | 2.2 Ω bootstrap charging resistor | 0805 SMD | Yageo | `RC0805FR-072R2L` |
| **C1, C4, C6** | 3 | 100 nF decoupling capacitor | 0805 SMD | Murata | `GRM21BR71H104KA01L` |
| **C2, C5, C7** | 3 | 1 µF supply capacitor | 0805 SMD | Murata | `GRM21BR71H105KA12L` |
| **C3** | 1 | 2.2 nF dead-time filter capacitor | 0805 SMD | Murata | `GRM21BR71H222KA01L` |
| **J3** | 1 | High-voltage terminal header, 1×03, 7.62 mm pitch | GMSTBA 1×03 | Phoenix Contact | `1766246` |
| **J1, J2** | 2 | Low-voltage headers, 1×02 and 1×04 | 2.54 mm pitch | Generic | 1×02 and 1×04 male headers |

---

## Design Limitations and Possible Improvements

The current board prioritises clear spacing and straightforward routing. As a result, some connections between the gate driver, MOSFETs, and `C8` are longer than they could be. Shortening these paths and using copper pours where appropriate could reduce stray inductance and switching-related ringing.

The bootstrap diode's estimated 1 V forward drop brings the high-side gate-drive voltage down to about 14 V. The report also notes that the simulated Miller-related gate bounce is around 0.8 V, while a larger spike could occur in real hardware. A small isolated bipolar supply providing −4 V and +15 V is one option to address both concerns.

Finally, the present dead time is 220 ns. Reducing `R1` from 22 kΩ to 10 kΩ would give approximately 100 ns and could reduce conduction losses during dead time. This change should be checked carefully to ensure that shoot-through remains prevented.
