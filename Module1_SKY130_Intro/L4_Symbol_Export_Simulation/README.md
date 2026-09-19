# L4 — Creating Symbol and Exporting Schematic in Xschem

## Objective

The objective of this lab was to convert the transistor-level CMOS inverter schematic into a reusable hierarchical symbol, create a separate testbench, generate a SPICE netlist, and functionally verify the inverter using **ngspice**.

The schematic was also configured to generate a hierarchical `.subckt` netlist for later **Layout Versus Schematic (LVS)** verification.

The overall flow was:

```text
CMOS Inverter Schematic
        ↓
Create Inverter Symbol
        ↓
Instantiate Symbol in Testbench
        ↓
Connect VDD, VIN and VSS
        ↓
Include SKY130 Device Models
        ↓
Configure Transient Analysis
        ↓
Generate SPICE Netlist
        ↓
Run ngspice Simulation
        ↓
Verify VIN / VOUT
        ↓
Prepare .subckt Netlist for LVS
```

---

## 1. Creating a Symbol from the Inverter Schematic

The CMOS inverter created in the previous lab contains a PMOS and NMOS with four external connections:

- `IN`
- `OUT`
- `VDD`
- `VSS`

Instead of recreating the transistor-level schematic every time the inverter is used, Xschem can generate a reusable symbol from the schematic.

From the inverter schematic, I selected:

```text
Symbol → Make symbol from schematic
```

![Creating symbol from schematic](images/01_create_symbol_from_schematic.png)

This generated:

```text
inverter.sym
```

The symbol provides a hierarchical representation of the transistor-level inverter.

---

## 2. Generated Inverter Symbol

The generated inverter symbol exposes the external terminals of the original schematic:

```text
IN
VDD
OUT
VSS
```

The symbol can now be instantiated inside another schematic while the actual PMOS/NMOS implementation remains inside `inverter.sch`.

![Generated inverter symbol](images/02_generated_inverter_symbol.png)

This introduced the concept of **hierarchical schematic design**:

```text
Testbench
   │
   └── inverter.sym
           │
           └── inverter.sch
                   │
                   ├── PMOS
                   └── NMOS
```

The symbol therefore acts as the interface to the underlying transistor-level circuit.

---

## 3. Creating the Inverter Testbench

A new Xschem schematic was created to functionally test the inverter.

The inverter symbol was instantiated in the testbench and voltage sources were added using:

```text
vsource.sym
```

![Adding voltage source](images/03_add_voltage_source.png)

Two voltage sources were required:

### Supply voltage

The SKY130 low-voltage MOS devices used in the inverter operate with a nominal supply of:

```text
VDD = 1.8 V
```

### Input stimulus

The second voltage source drives the inverter input using a Piecewise Linear (PWL) waveform:

```spice
PWL(0 0 20n 0 900n 1.8)
```

This means:

```text
0 ns      → 0 V
20 ns     → 0 V
900 ns    → 1.8 V
```

Therefore, the input initially remains at 0 V and then gradually ramps toward 1.8 V.

This allows the switching behavior of the CMOS inverter to be observed during transient simulation.

---

## 4. Including the SKY130 ngspice Device Models

The transistor symbols in the schematic describe the circuit connectivity, but ngspice also requires the electrical device models supplied by the SKY130 PDK.

A `code_shown.sym` block was added with:

```spice
.lib /usr/share/pdk/sky130A/libs.tech/ngspice/sky130.lib.spice tt
```

![SKY130 ngspice model library](images/04_sky130_model_library.png)

The `.lib` statement loads the SKY130 transistor model library.

The:

```text
tt
```

option selects the **Typical-Typical (TT)** process corner for the simulation.

---

## 5. Configuring Transient Analysis

A second `code_shown.sym` block was used to define the ngspice transient simulation.

```spice
.control
tran 1n 1u
plot V(IN) V(OUT)
.endc
```

![Transient simulation control block](images/05_transient_control_setup.png)

### Simulation parameters

`tran 1n 1u` specifies:

```text
Simulation step = 1 ns
Stop time       = 1 µs
```

The following command plots both the inverter input and output:

```spice
plot V(IN) V(OUT)
```

This allows the input transition and corresponding inverter response to be viewed together.

---

## 6. Completed Inverter Testbench

The completed testbench contains:

- Hierarchical inverter symbol
- 1.8 V VDD source
- PWL input voltage source
- Ground connection
- `IN` net
- `OUT` net
- SKY130 ngspice model library
- Transient simulation commands

![Complete inverter testbench](images/06_complete_inverter_testbench.png)

The resulting simulation flow is:

```text
PWL Voltage Source
       │
       ▼
      IN
       │
       ▼
 ┌───────────┐
 │  Inverter │
 └───────────┘
       │
       ▼
      OUT
       │
       ▼
Transient Waveform
```

---

## 7. Functional Verification Using ngspice

After completing the testbench, the schematic was netlisted and simulated using ngspice.

The final transient simulation successfully produced both:

```text
V(IN)
V(OUT)
```

![Inverter transient simulation result](images/07_transient_simulation_result.png)

### Simulation Result

The red waveform represents:

```text
V(IN)
```

and gradually increases from approximately:

```text
0 V → 1.8 V
```

The blue waveform represents:

```text
V(OUT)
```

and initially remains near:

```text
1.8 V
```

When the input enters the inverter switching region, the output rapidly transitions toward:

```text
0 V
```

Therefore:

```text
VIN LOW  → VOUT HIGH
VIN HIGH → VOUT LOW
```

This confirms the expected logical behavior of the CMOS inverter.

---

## 8. Simulation Debugging

During the initial simulation, transient analysis completed successfully, but ngspice reported:

```text
Error: no such vector out
```

The generated node list initially contained automatically generated nodes such as:

```text
net1
net2
```

instead of the expected:

```text
IN
OUT
```

`net1` was identified as the VDD rail because it remained at 1.8 V.

This indicated that the issue was not with the transient analysis itself, but with the hierarchical net/pin connectivity between the inverter schematic, generated symbol, and testbench.

After checking the symbol connectivity and correcting the testbench net connections, the netlist correctly recognized:

```text
in
out
```

The simulation then successfully generated both input and output waveforms.

### Debugging Takeaway

A schematic can appear visually connected while the generated SPICE netlist does not contain the expected node names.

When debugging this type of issue, checking the **ngspice node list and generated netlist** helps distinguish between:

- simulation-command errors,
- model errors,
- and schematic/net connectivity errors.

---

## 9. Preparing the Schematic for LVS

After functional verification, the inverter schematic was configured for later Layout Versus Schematic verification.

In Xschem:

```text
Simulation → LVS netlist: Top level is a .subckt
```

was enabled.

![LVS top-level subcircuit setting](images/08_lvs_subckt_setting.png)

With this option enabled, Xschem generates the top-level schematic as a SPICE subcircuit.

Conceptually, the resulting netlist contains:

```spice
.subckt inverter ...
...
.ends inverter
```

rather than treating the inverter as a complete standalone simulation circuit.

### Why `.subckt` is important

A subcircuit provides a reusable hierarchical representation of the inverter.

This becomes important during LVS because the schematic representation of the inverter can be compared against the corresponding netlist extracted from its physical layout.

It also allows the inverter to be instantiated as a building block inside larger designs.

---

## Key Results

- Created a reusable hierarchical symbol from the transistor-level CMOS inverter schematic.
- Instantiated the inverter symbol inside a separate Xschem testbench.
- Configured a **1.8 V** supply for the SKY130 low-voltage devices.
- Applied a **PWL input stimulus** from 0 V to 1.8 V.
- Loaded the SKY130 ngspice models using the **TT process corner**.
- Configured and executed transient analysis using ngspice.
- Successfully observed complementary `V(IN)` and `V(OUT)` waveforms.
- Debugged a hierarchical output-net connectivity issue using the ngspice node list.
- Configured Xschem to export the inverter as a top-level `.subckt` for subsequent LVS verification.

---

## Final Result

The CMOS inverter was successfully converted into a reusable hierarchical block and functionally validated before physical layout implementation.

```text
Schematic
   ✓
Symbol
   ✓
Testbench
   ✓
SPICE Netlist
   ✓
Transient Simulation
   ✓
Functional Verification
   ✓
LVS-ready .subckt
   ✓
```

The next stage is to use the generated schematic/netlist information for physical layout and verification in Magic.
