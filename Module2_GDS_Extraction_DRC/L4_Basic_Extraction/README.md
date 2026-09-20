# L4 — Basic Extraction and Parasitic RC Netlist Generation

## Overview

In this lab, I explored the extraction flow in Magic using the SKY130 standard-cell layout.

The objective was to move from a physical layout representation to progressively more detailed electrical netlists:

1. Basic extracted connectivity
2. LVS-oriented SPICE netlist
3. SPICE netlist with parasitic capacitances
4. Resistance extraction
5. Final RC-aware SPICE netlist

The lab demonstrated how the same physical layout can be converted into different levels of electrical abstraction depending on whether the goal is LVS comparison or detailed post-layout simulation.

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
```

---

## 1. Extracting the Layout

I loaded the SKY130 standard cell used in the previous exercises and performed extraction using:

```tcl
extract all
```

Magic generated an intermediate extraction file:

```text
sky130_fd_sc_hd__and2_1.ext
```

I then configured the SPICE conversion for LVS and generated the netlist:

```tcl
ext2spice lvs
ext2spice
```

![Basic layout extraction](images/01_basic_layout_extraction.png)

After extraction, the working directory contained both:

```text
sky130_fd_sc_hd__and2_1.ext
sky130_fd_sc_hd__and2_1.spice
```

The `.ext` file is Magic's intermediate extracted representation, while the `.spice` file is the electrical netlist used by downstream tools.

---

## 2. LVS-Oriented SPICE Netlist

I inspected the generated SPICE file:

```bash
vi sky130_fd_sc_hd__and2_1.spice
```

![LVS extracted SPICE netlist](images/02_lvs_extracted_spice_netlist.png)

The extracted netlist contained the standard-cell subcircuit and the recognized transistor instances.

The top-level definition retained the port information:

```spice
.subckt sky130_fd_sc_hd__and2_1 A B VGND VNB VPB VPWR X
```

The extracted device network contained six transistor subcircuit calls.

This LVS-oriented version primarily represents:

```text
Devices
+
Electrical connectivity
```

without including the detailed parasitic RC network.

This makes it appropriate for schematic-vs-layout comparison.

---

## 3. Adding Parasitic Capacitance

The extraction database already contained capacitance information, but the earlier:

```tcl
ext2spice lvs
```

configuration did not include those parasitics in the SPICE output.

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

Magic included all extracted capacitances, including extremely small values.

The SPICE netlist now contained lines beginning with:

```text
C
```

such as:

```text
C0 ...
C1 ...
C2 ...
```

These represent layout-derived parasitic capacitances between physical nets.

---

## 4. Applying a Capacitance Threshold

Some extracted capacitances were extremely small and rounded close to zero.

To remove insignificant values, I changed the capacitance threshold to:

```tcl
ext2spice cthresh 0.01
ext2spice
```

![Capacitance threshold](images/04_capacitance_threshold.png)

The resulting netlist retained only capacitances above the selected threshold.

Conceptually:

```text
cthresh 0
    ↓
Include every extracted capacitance

cthresh 0.01
    ↓
Remove very small capacitances
```

This produces a cleaner parasitic netlist while keeping the more relevant capacitive effects.

---

## 5. Preparing Data for Resistance Extraction

Parasitic resistance extraction requires additional connectivity information.

I generated the intermediate simulation data using:

```tcl
ext2sim labels on
ext2sim
```

![ext2sim intermediate files](images/05_ext2sim_intermediate_files.png)

This created additional files including:

```text
sky130_fd_sc_hd__and2_1.nodes
sky130_fd_sc_hd__and2_1.sim
```

These files provide intermediate network information used by the resistance-extraction flow.

The `.sim` representation can also be associated with switch-level simulation flows, while here it serves as part of the extraction pipeline.

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

The tolerance controls when a network is detailed enough to require replacement with a distributed resistance representation.

A value of:

```text
10
```

was used in this exercise.

During extraction, Magic identified ports and created drive points for the corresponding nets.

The output reported information such as:

```text
Total Nets
Nets extracted
Nets output
```

showing how many networks were analyzed and how many were converted into resistance networks.

---

## 7. Resistance Extraction Output

After running `extresist`, a new file appeared in the working directory:

```text
sky130_fd_sc_hd__and2_1.res.ext
```

![Resistance extraction file](images/07_resistance_extraction_file.png)

This file does not replace the original `.ext` database.

Instead, it contains the resistance information required to modify selected extracted networks when the final SPICE netlist is generated.

The extraction flow therefore becomes:

```text
Original .ext
     +
.res.ext
     ↓
ext2spice
     ↓
RC-aware SPICE netlist
```

---

## 8. Generating the Final RC-Aware Netlist

For the final extraction, I configured `ext2spice` with:

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

which instructs the SPICE converter to incorporate the resistance-extraction data into the output netlist.

The final SPICE file therefore contains:

```text
Transistor devices
+
Parasitic capacitances
+
Parasitic resistances
```

This is a much more physically realistic representation of the layout than the original LVS-only netlist.

---

## Extraction Levels Compared

The lab produced several increasingly detailed circuit representations.

| Extraction Level | Devices | Capacitance | Resistance | Main Use |
|---|---:|---:|---:|---|
| LVS netlist | ✓ | — | — | Layout vs. schematic comparison |
| Capacitive netlist | ✓ | ✓ | — | Post-layout simulation |
| RC netlist | ✓ | ✓ | ✓ | Detailed post-layout analysis |

The progression can be represented as:

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

## Understanding the Extracted Resistance Network

Resistance extraction breaks a physical conductor into a distributed network instead of treating an entire wire as one ideal node.

Conceptually:

```text
Ideal wire:

A ───────────────── OUT
```

becomes something closer to:

```text
A ─R1─●─R2─●─R3─ OUT
      │     │
     C1    C2
      │     │
     GND   GND
```

This is important because real interconnect has both:

```text
Resistance
Capacitance
```

which affect delay and waveform shape.

---

## Extracted Internal Node Naming

The detailed resistance network creates additional internal nodes.

The extraction process distinguishes locations such as:

- branch points in a network
- wire termination points
- device connection points

These additional nodes allow the original physical interconnect to be represented as a distributed electrical network rather than as one ideal connection.

As the extracted representation becomes more detailed, the SPICE netlist becomes considerably larger.

---

## Why RC Extraction Matters

The schematic assumes wires are ideal:

```text
Rwire = 0
Cwire = 0
```

The physical layout does not behave this way.

Real interconnect introduces:

```text
Metal resistance
+
Coupling capacitance
+
Capacitance to surrounding structures
```

These parasitics can affect:

- propagation delay
- rise/fall time
- switching behavior
- analog settling
- timing accuracy

Therefore, RC-aware extraction allows simulation to better represent the manufactured circuit.

---

## LVS vs. Post-Layout Extraction

An important distinction from this lab is:

### LVS extraction

The objective is primarily connectivity verification.

```text
Layout
 ↓
Devices + nets
 ↓
Compare against schematic
```

### Post-layout extraction

The objective is circuit behavior including physical parasitics.

```text
Layout
 ↓
Devices + nets + parasitic C + parasitic R
 ↓
SPICE simulation
```

The same layout can therefore produce different netlists depending on the verification goal.

---

## Key Commands

### Basic Extraction

```tcl
extract all
```

### LVS SPICE Netlist

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

### Generate Intermediate Simulation Data

```tcl
ext2sim labels on
ext2sim
```

### Resistance Extraction

```tcl
extresist tolerance 10
extresist
```

### Final RC SPICE Netlist

```tcl
ext2spice lvs
ext2spice cthresh 0.01
ext2spice extresist on
ext2spice
```

---

## Generated Files

During the extraction flow I generated several representations:

```text
sky130_fd_sc_hd__and2_1.ext
sky130_fd_sc_hd__and2_1.spice
sky130_fd_sc_hd__and2_1.nodes
sky130_fd_sc_hd__and2_1.sim
sky130_fd_sc_hd__and2_1.res.ext
```

Their roles can be summarized as:

| File | Purpose |
|---|---|
| `.ext` | Magic extracted circuit database |
| `.spice` | SPICE netlist |
| `.nodes` | Extracted-node information |
| `.sim` | Intermediate simulation/extraction representation |
| `.res.ext` | Distributed resistance extraction information |

---

## Key Learnings

- Extracted electrical connectivity directly from a SKY130 physical layout.
- Converted Magic `.ext` extraction data into a SPICE netlist.
- Generated an LVS-oriented netlist without parasitic RC detail.
- Added layout-derived parasitic capacitance using `ext2spice`.
- Used `cthresh` to remove insignificant extracted capacitances.
- Generated `.nodes` and `.sim` intermediate extraction files.
- Performed distributed interconnect resistance extraction with `extresist`.
- Generated a `.res.ext` resistance representation.
- Incorporated resistance data into the final SPICE netlist.
- Understood the difference between LVS extraction and post-layout RC extraction.
- Connected physical interconnect geometry with its equivalent electrical RC network.

---

## Result

```text
Layout extraction completed          ✓
.ext database generated              ✓
LVS SPICE netlist generated          ✓
Device connectivity verified         ✓
Parasitic capacitances extracted     ✓
Capacitance threshold applied        ✓
ext2sim data generated               ✓
Resistance extraction completed      ✓
.res.ext generated                   ✓
RC-aware SPICE netlist generated     ✓
```

The lab demonstrated the complete progression from a physical SKY130 standard-cell layout to an electrical netlist containing devices and layout-derived parasitic RC information.

---

## Suggested Follow-Up

A useful next experiment is to simulate the same cell using three different representations:

```text
1. LVS / ideal extracted netlist

2. Extracted netlist
   + parasitic capacitance

3. Extracted netlist
   + parasitic capacitance
   + parasitic resistance
```

Comparing the transient outputs would show directly how physical-layout parasitics affect circuit timing and waveform behavior.
