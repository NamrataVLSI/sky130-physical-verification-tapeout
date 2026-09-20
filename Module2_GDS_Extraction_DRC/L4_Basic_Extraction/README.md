# L4 — Basic Extraction and Parasitic RC Netlist Generation

## Overview

In this lab, I explored the extraction flow in Magic using a SKY130 standard-cell layout.

The goal was to move from the physical layout to progressively more detailed electrical representations:

1. Basic extracted connectivity
2. LVS-oriented SPICE netlist
3. SPICE netlist with parasitic capacitances
4. Resistance extraction
5. Final RC-aware SPICE netlist

This lab demonstrated that the same physical layout can be extracted differently depending on whether the objective is **LVS verification** or **post-layout electrical simulation**.

The overall flow was:

```text
SKY130 Standard-Cell Layout
        ↓
Magic Extraction
        ↓
.ext Intermediate File
        ↓
LVS SPICE Netlist
        ↓
Add Parasitic Capacitance
        ↓
Generate ext2sim Data
        ↓
Extract Interconnect Resistance
        ↓
.res.ext Resistance Data
        ↓
Generate Final RC SPICE Netlist
        ↓
Inspect Extracted R + C Network
```

---

## 1. Basic Layout Extraction

I loaded the SKY130 standard cell used in the previous exercises and extracted its electrical connectivity using:

```tcl
extract all
```

Magic generated an intermediate extraction file:

```text
sky130_fd_sc_hd__and2_1.ext
```

I then configured the SPICE output for LVS and generated the netlist:

```tcl
ext2spice lvs
ext2spice
```

![Basic layout extraction](images/01_basic_layout_extraction.png)

The working directory now contained:

```text
sky130_fd_sc_hd__and2_1.ext
sky130_fd_sc_hd__and2_1.spice
```

The `.ext` file is Magic's intermediate extracted representation, while the `.spice` file is the electrical netlist generated from it.

---

## 2. LVS-Oriented SPICE Netlist

I inspected the generated SPICE file using:

```bash
vi sky130_fd_sc_hd__and2_1.spice
```

![LVS extracted SPICE netlist](images/02_lvs_extracted_spice_netlist.png)

The extracted subcircuit retained the standard-cell port order:

```spice
.subckt sky130_fd_sc_hd__and2_1 A B VGND VNB VPB VPWR X
```

The netlist contained six transistor subcircuit instances corresponding to the devices recognized from the physical layout.

At this stage, the main information represented was:

```text
Devices
+
Electrical connectivity
```

without detailed parasitic RC components.

This type of netlist is suitable for **Layout Versus Schematic (LVS)** comparison.

---

## 3. Adding Parasitic Capacitance

The extraction database already contained capacitance information, but the LVS-oriented output did not include it in the SPICE netlist.

To include all extracted capacitances, I used:

```tcl
ext2spice cthresh 0
ext2spice
```

![All parasitic capacitances](images/03_all_parasitic_capacitances.png)

With:

```text
cthresh = 0
```

Magic included all extracted parasitic capacitances.

The SPICE file now contained entries beginning with:

```text
C0
C1
C2
...
```

representing capacitance between physical nets in the layout.

---

## 4. Applying a Capacitance Threshold

Some extracted capacitances were extremely small and rounded very close to zero.

To remove insignificant values, I changed the threshold to:

```tcl
ext2spice cthresh 0.01
ext2spice
```

![Capacitance threshold](images/04_capacitance_threshold.png)

The behavior can be summarized as:

```text
cthresh 0
    ↓
Include all extracted capacitances

cthresh 0.01
    ↓
Exclude very small capacitances
```

This creates a cleaner parasitic netlist while retaining the more relevant capacitive effects.

---

## 5. Preparing Data for Resistance Extraction

Resistance extraction requires additional intermediate connectivity information.

I generated this using:

```tcl
ext2sim labels on
ext2sim
```

![ext2sim intermediate files](images/05_ext2sim_intermediate_files.png)

This generated files including:

```text
sky130_fd_sc_hd__and2_1.nodes
sky130_fd_sc_hd__and2_1.sim
```

These files contain intermediate network information used by the resistance-extraction flow.

---

## 6. Extracting Interconnect Resistance

I configured resistance extraction using:

```tcl
extresist tolerance 10
```

and then ran:

```tcl
extresist
```

![Resistance extraction](images/06_resistance_extraction.png)

The tolerance determines when an extracted net should be replaced with a distributed resistance network.

For this exercise, I used:

```text
10
```

Magic identified the circuit ports and generated drive points for the extracted networks.

The console reported information such as:

```text
Total Nets
Nets extracted
Nets output
```

showing how many nets were analyzed and how many were converted into resistance networks.

---

## 7. Resistance Extraction Output File

After running `extresist`, a new file appeared:

```text
sky130_fd_sc_hd__and2_1.res.ext
```

![Resistance extraction file](images/07_resistance_extraction_file.png)

This file contains resistance information associated with the original extraction database.

The flow now becomes:

```text
Original .ext
      +
.res.ext
      ↓
ext2spice
      ↓
Final RC-aware SPICE netlist
```

The `.res.ext` file is therefore used to augment the original extraction data with distributed interconnect resistance.

---

## 8. Generating the Final RC-Aware SPICE Netlist

I configured the final SPICE extraction using:

```tcl
ext2spice lvs
ext2spice cthresh 0.01
ext2spice extresist on
ext2spice
```

![Final RC SPICE generation](images/08_final_rc_spice_generation.png)

The important additional option was:

```tcl
ext2spice extresist on
```

which instructs Magic to include the resistance-extraction data in the generated SPICE netlist.

The final netlist therefore includes:

```text
Transistor devices
+
Parasitic capacitances
+
Parasitic resistances
```

This is a more physically realistic representation of the implemented standard cell.

---

## 9. Inspecting the Final RC-Extracted SPICE Netlist

After generating the RC-aware netlist, I inspected the final SPICE file.

![Final RC extracted SPICE netlist](images/09_final_rc_extracted_netlist.png)

The final netlist now clearly contained three major element groups.

### Device Instances

Lines beginning with:

```text
X
```

represent the extracted transistor/device instances.

For example:

```spice
X0 ...
X1 ...
X2 ...
```

These correspond to the MOS devices recognized from the physical layout.

### Parasitic Resistances

Lines beginning with:

```text
R
```

represent distributed interconnect resistance.

Examples included entries such as:

```spice
R0 B.n0 B.t0 ...
R1 B.n0 B.t1 ...
R3 VPWR.n0 VPWR.t1 ...
R8 X X.t1 ...
```

The original ideal nets were divided into internal nodes such as:

```text
B.n0
B.t0
B.t1
VPWR.n0
VPWR.t0
X.n1
X.t1
```

This represents the physical interconnect as a distributed resistive network instead of one ideal zero-resistance node.

### Parasitic Capacitances

The same netlist also contained:

```text
C0
C1
C2
...
```

for the extracted parasitic capacitances.

Examples included coupling between nets such as:

```text
A ↔ X
B ↔ X
A ↔ VPWR
VGND ↔ A
X ↔ VPWR
```

The final extracted representation therefore contains:

```text
Devices
+
Distributed R
+
Parasitic C
```

which is suitable for more realistic post-layout simulation.

---

## 10. Extraction Levels Compared

This lab generated several progressively more detailed representations of the same layout.

| Extraction Level | Devices | Parasitic C | Parasitic R | Main Purpose |
|---|---:|---:|---:|---|
| LVS netlist | ✓ | — | — | Connectivity verification |
| Capacitive netlist | ✓ | ✓ | — | Post-layout simulation |
| RC-aware netlist | ✓ | ✓ | ✓ | More detailed post-layout analysis |

The progression can be visualized as:

```text
Ideal Connectivity
       ↓
Devices
       ↓
Devices + C
       ↓
Devices + C + R
       ↓
More realistic physical behavior
```

---

## 11. Understanding the Distributed RC Network

In the ideal schematic, a wire is treated as one electrical node:

```text
A ───────────────── OUT
```

But a physical interconnect has resistance and capacitance.

The extracted representation is closer to:

```text
A ─R1─●─R2─●─R3─ OUT
      │     │
     C1    C2
      │     │
     GND   GND
```

The resistor network captures voltage drop and distributed interconnect behavior, while the capacitances represent coupling and loading caused by the physical geometry.

---

## 12. Internal Node Naming

Resistance extraction introduces additional internal node names.

Examples from the extracted netlist included:

```text
B.n0
B.t0
VPWR.n0
VPWR.t1
X.n1
X.t1
A.n0
A.t0
```

These internal nodes allow the original physical network to be represented as multiple resistor segments.

The detailed extracted network is therefore substantially larger than the original ideal schematic netlist.

---

## 13. Why RC Extraction Matters

An ideal schematic generally assumes:

```text
Rwire = 0
Cwire = 0
```

Real physical interconnect introduces:

```text
Metal / poly resistance
+
Capacitance to neighboring nets
+
Capacitance to substrate
```

These parasitic effects can influence:

- propagation delay
- rise and fall times
- analog settling
- signal integrity
- transient response
- timing accuracy

Therefore, RC-aware extraction provides a much better representation of the manufactured circuit than the schematic alone.

---

## 14. LVS Extraction vs. Post-Layout Extraction

An important distinction from this lab is the purpose of each extraction mode.

### LVS-Oriented Extraction

For LVS, the main concern is:

```text
Are the devices and connections in the layout
electrically equivalent to the schematic?
```

The flow is:

```text
Layout
  ↓
Devices + Connectivity
  ↓
LVS Comparison
```

### Post-Layout Extraction

For post-layout simulation, the physical parasitics are also important:

```text
Layout
  ↓
Devices
+ Resistance
+ Capacitance
  ↓
SPICE Simulation
```

The same physical layout can therefore produce different netlists depending on the intended verification task.

---

## Generated Files

During this lab, the extraction flow generated:

```text
sky130_fd_sc_hd__and2_1.ext
sky130_fd_sc_hd__and2_1.spice
sky130_fd_sc_hd__and2_1.nodes
sky130_fd_sc_hd__and2_1.sim
sky130_fd_sc_hd__and2_1.res.ext
```

Their roles are:

| File | Purpose |
|---|---|
| `.ext` | Magic's extracted electrical database |
| `.spice` | SPICE netlist generated from extraction |
| `.nodes` | Extracted node information |
| `.sim` | Intermediate simulation/extraction representation |
| `.res.ext` | Resistance extraction information |

---

## Key Commands

### Basic Extraction

```tcl
extract all
```

### LVS-Oriented SPICE

```tcl
ext2spice lvs
ext2spice
```

### Include All Capacitances

```tcl
ext2spice cthresh 0
ext2spice
```

### Apply Capacitance Threshold

```tcl
ext2spice cthresh 0.01
ext2spice
```

### Generate Intermediate Data

```tcl
ext2sim labels on
ext2sim
```

### Resistance Extraction

```tcl
extresist tolerance 10
extresist
```

### Generate Final RC Netlist

```tcl
ext2spice lvs
ext2spice cthresh 0.01
ext2spice extresist on
ext2spice
```

---

## Key Learnings

- Extracted electrical connectivity from a SKY130 standard-cell layout.
- Generated Magic `.ext` extraction data.
- Converted the extracted layout into a SPICE netlist.
- Generated an LVS-oriented representation containing devices and connectivity.
- Added layout-derived parasitic capacitance.
- Applied a capacitance threshold to remove insignificant values.
- Generated `.nodes` and `.sim` intermediate extraction data.
- Performed distributed resistance extraction using `extresist`.
- Generated a `.res.ext` file containing resistance information.
- Integrated extracted resistance into the final SPICE netlist.
- Inspected the final resistor and capacitor networks directly.
- Understood how ideal nets become distributed RC networks after extraction.
- Distinguished LVS extraction from detailed post-layout extraction.

---

## Result

```text
Layout extraction completed          ✓
.ext database generated              ✓
LVS SPICE netlist generated          ✓
Device connectivity extracted        ✓
Parasitic capacitances extracted     ✓
Capacitance threshold applied        ✓
ext2sim intermediate files created   ✓
Resistance extraction completed      ✓
.res.ext file generated              ✓
RC-aware SPICE netlist generated     ✓
Final R/C network inspected          ✓
```

The lab demonstrated the complete progression from a physical SKY130 layout to an electrical SPICE representation containing device connectivity and layout-derived parasitic resistance and capacitance.

---

## Suggested Follow-Up

A useful follow-up experiment is to simulate the same logic cell with three different netlists:

```text
1. Ideal / LVS extracted netlist

2. Device netlist
   + parasitic capacitance

3. Device netlist
   + parasitic capacitance
   + parasitic resistance
```

Comparing the transient waveforms from these three representations would directly show the effect of physical-layout parasitics on circuit delay and waveform behavior.
