# 400 V SiC MOSFET Isolated Half-Bridge Gate Driver PCB

![PCB 3D Render](Media/PCB_3D_Render.png)

## Overview
This repository contains the hardware design, KiCad source files, manufacturing outputs, and LTspice verification for a **400 V DC isolated half-bridge power stage**. The design drives two **Wolfspeed 900 V Silicon Carbide (SiC) MOSFETs** using the **Texas Instruments UCC21520DW** dual-channel isolated driver.

The project explores layout considerations for wide-bandgap (WBG) power semiconductors, high-voltage spacing under IPC-2221B standards, asymmetric gate drive design, and transient verification in LTspice.

📄 **[Read the Full Technical Report (PDF)](Documentation/Technical Report - 400V SiC MOSFET Isolated Half-Bridge Gate Driver PCB.pdf)**

---

## Key Specifications
* **DC Bus Voltage:** 400 V DC nominal (900 V switch breakdown margin)
* **Power Switches:** 2× Wolfspeed `C3M0120090D` SiC MOSFETs (900 V, 23 A, R_DS(on) = 120 mΩ, TO-247-3)
* **Gate Driver:** Texas Instruments `UCC21520DW` (5.7 kVrms reinforced isolation, 4 A source / 6 A sink, SOIC-16W)
* **Gate Drive Supply:** Unipolar +15 V auxiliary rail with floating ultrafast bootstrap network (`UF4007`)
* **Hardware Dead-Time:** 220 ns interlock delay programmed via R_DT = 22 kΩ
* **High-Voltage Clearance:** Custom KiCad DRC rule enforcing **2.9 mm minimum clearance** on 400 V nets (`DC_PLUS`, `SW_NODE`), exceeding IPC-2221B bare-board criteria (2.5 mm)

---

## Hardware Architecture & PCB Layout

### 1. Schematic Design
![Schematic](Media/Schematic.png)
* **Galvanic Isolation:** Isolates the 5 V logic input domain (`GND`) from the 400 V switching power ground (`GNDPWR`) and midpoint (`SW_NODE`).
* **Asymmetric Gate Drive:** Split turn-on (10 Ω) and turn-off (1 Ω || 10 Ω ≈ 0.91 Ω via `1N5819HW` Schottky diode) network to moderate turn-on di/dt while clamping the gate tightly to 0 V to resist Miller-induced false turn-on.
* **Series Bleeder Network:** Two 220 kΩ 0207 metal-film resistors in series divide the 400 V bus to 200 V and ~0.18 W per device, remaining within individual 250 V / 0.25 W limits.

### 2. Two-Layer PCB Layout
![2D Layout](Media/PCB_2D_Layout.png)
* **Isolation Keep-Out:** Continuous copper keep-out maintained beneath `U1` across both layers, preserving >8 mm creepage.
* **TO-247 Pad Neck-Downs:** High-current 2.5 mm power traces neck down to 1.5 mm at `Q1` and `Q2` pads, exiting at 90° angles to maintain safe spacing within the package-limited 5.45 mm pin pitch.
* **Top-Layer Switching Node:** `SW_NODE` is routed entirely on the top copper layer (`F.Cu`) without layer-transition vias to minimize stray loop inductance.

---

## LTspice Transient Verification

The secondary-side half-bridge switching cell was modeled and verified in LTspice 26 over a 20 µs window (two 100 kHz periods).

![LTspice Schematic](Media/LTspice_Schematic.png)

![LTspice Waveforms](Media/LTspice_Waveforms_Full.png)

### Key Simulation Findings
1. **Switching Integrity:** `SW_NODE` transitions rail-to-rail (0 V to 400 V) with controlled turn-off overshoot (clamped by local 100 nF snubber `C8`).
2. **Floating High-Side Drive:** Differential gate drive `V(HS_GATE, SW_NODE)` delivers a stable 0 V to 14 V pulse, accounting for the ~1.0 V diode drop across `D1`.
3. **Miller Spike Suppression:** During 400 V turn-on transitions, low-side gate bounce peaks at ~0.78 V, providing a safe margin below the 1.8 V minimum threshold of the `C3M0120090D`.

---

## Bill of Materials (BOM)

| Ref | Qty | Value / Role | Package | Manufacturer | Part Number (MPN) |
| :--- | :-: | :--- | :--- | :--- | :--- |
| **U1** | 1 | Isolated Gate Driver | SOIC-16W | Texas Instruments | `UCC21520DW` |
| **Q1, Q2** | 2 | 900V SiC MOSFET | TO-247-3 | Wolfspeed | `C3M0120090D` |
| **D1** | 1 | Bootstrap Diode | DO-41 Axial | Generic | `UF4007` |
| **D2, D3** | 2 | Turn-Off Diode | SOD-123 | Generic | `1N5819HW` |
| **C8** | 1 | 100nF Snubber (630V) | Radial (P=15mm) | WIMA | `MKP1J031004B00KSSD` |
| **R9, R10**| 2 | 220kΩ Bleeder (250V)| 0207 Axial | Yageo | `MFR-25FTE52-220K` |
| **R1** | 1 | 22kΩ Dead-Time | 0805 SMD | Yageo | `RC0805FR-0722KL` |
| **R2, R5** | 2 | 10Ω Turn-On | 0805 SMD | Yageo | `RC0805FR-0710RL` |
| **R3, R6** | 2 | 1Ω Turn-Off | 0805 SMD | Yageo | `RC0805FR-071RL` |
| **R4, R7** | 2 | 10kΩ Pull-Down | 0805 SMD | Yageo | `RC0805FR-0710KL` |
| **R8** | 1 | 2.2Ω Inrush Limit | 0805 SMD | Yageo | `RC0805FR-072R2L` |
| **C1, C4, C6**| 3 | 100nF Decoupling | 0805 SMD | Murata | `GRM21BR71H104KA01L` |
| **C2, C5, C7**| 3 | 1µF Bulk Supply | 0805 SMD | Murata | `GRM21BR71H105KA12L` |
| **C3** | 1 | 2.2nF DT Filter | 0805 SMD | Murata | `GRM21BR71H222KA01L` |
| **J3** | 1 | HV Power Terminal | GMSTBA 1x03 | Phoenix Contact | `1766246` *(Plug: `1767012`)* |
| **J1, J2** | 2 | Low-Voltage Headers | 2.54mm Pitch | Generic | `1x02` & `1x04` Male |

---

## Improvement Roadmap
* **Bipolar Gate Drive (-4 V / +15 V):** Integrate an isolated DC-DC converter to provide negative off-state bias for higher temperature-dependent Miller immunity.
* **Bootstrap Diode Compensation:** Upgrade `D1` to a low-drop SiC Schottky diode to bring high-side gate overdrive up to the full nominal 15 V level.
* **Dead-Time Trimming:** Reduce `R1` to 10 kΩ (~100 ns delay) to minimize conduction losses across the 4.2 V SiC body diode.
* **4-Layer PCB Stackup:** Implement an internal ground reference plane directly beneath the switching loop to minimize trace inductance via magnetic flux cancellation.
