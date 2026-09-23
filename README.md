# SKY130 Physical Design & Verification

This repository documents my hands-on work with the **SkyWater SKY130 PDK and open-source EDA tools**, covering the IC design flow from device-level layout and physical verification to RTL-to-GDSII implementation and LVS debugging.

The goal of this project was not just to run the tools and collect passing results. I wanted to understand what happens at each stage, what the verification reports actually mean, and how to trace a failure back to the layout, schematic, netlist, or device property responsible for it.

The overall learning path was:

```text
SKY130 & Device Layout
          ↓
GDS / Extraction / Physical Verification
          ↓
Detailed DRC Rule Debugging
          ↓
RTL-to-GDSII with OpenLane
          ↓
LVS & Debugging
```

---

## 🛠️ Tools & Technologies

| Area | Tools / Technology |
|---|---|
| Process Technology | SkyWater SKY130 PDK |
| Layout | Magic VLSI |
| Schematic Capture | Xschem |
| Circuit Simulation | Ngspice |
| LVS | Netgen |
| Digital Physical Design | OpenLane |
| Synthesis | Yosys + ABC |
| Static Timing Analysis | OpenSTA |
| Clock Tree Synthesis | TritonCTS |
| Global Routing | FastRoute |
| Detailed Routing | TritonRoute |
| Physical Verification | Magic, KLayout, Netgen, CVC |
| Layout Exchange | GDSII, LEF, DEF |
| Environment | Linux, Tcl, Shell |

---

# 🧭 Repository Journey

Each module builds on the previous one.

Rather than treating layout, DRC, extraction, digital implementation, and LVS as independent topics, this repository follows how they connect in an actual physical-design flow.

---

# 01 — Building the Foundation with SKY130

## Module 1 — SKY130 Introduction & Device-Level Flow

The first module introduced the **SKY130 PDK and the basic schematic-to-layout workflow**.

I started by understanding the technology environment and then worked through creating and validating a simple CMOS design using **Xschem and Magic**.

The work included:

- Understanding the SKY130 PDK environment
- Creating a CMOS inverter schematic in Xschem
- Creating reusable schematic symbols
- Building a testbench
- Running SPICE simulation
- Importing schematic-generated SPICE into Magic
- Creating and modifying SKY130 device layouts
- Extracting layout netlists
- Running an initial LVS flow

This module established the connection between:

```text
Schematic
    ↓
Simulation
    ↓
Layout
    ↓
Extraction
    ↓
LVS
```

It provided the foundation for the more detailed physical-verification work in the following modules.

---

# 02 — Understanding What Is Inside a Layout Database

## Module 2 — GDS, Extraction, DRC, LVS & XOR

The second module moved deeper into the layout database itself.

Instead of only drawing geometry, I explored how physical-design information is represented across **GDS, LEF, SPICE, and Magic's internal database**.

The labs covered:

- Reading SKY130 GDS libraries into Magic
- GDS/CIF input styles
- Vendor layout handling
- Port names and port indices
- Port metadata
- LEF annotation
- SPICE-based port ordering
- Abstract views
- Read-only library cells
- Layout extraction
- Parasitic capacitance extraction
- Parasitic resistance extraction
- RC-aware SPICE generation
- Batch and interactive DRC
- Hierarchical DRC
- Basic LVS setup with Netgen
- XOR-based layout comparison

One of the important lessons from this module was that different file formats preserve different information.

```text
GDS
→ Physical geometry

LEF
→ Abstract physical information + pin metadata

SPICE
→ Electrical connectivity + device information
```

Understanding those differences became especially important later when debugging hierarchical LVS.

The module ended with XOR comparison, where I used Magic to compare two physical layouts and isolate only the geometry that changed.

---

# 03 — When the Layout Breaks the Rules

## Module 3 — SKY130 Design Rule Checking

Once I was comfortable navigating and extracting layouts, I moved into detailed **Design Rule Checking (DRC)**.

Instead of simply running DRC and looking for a green checkmark, these labs intentionally introduced violations so I could understand:

> What rule was violated, why the foundry requires that rule, and what geometry must change to fix it?

The labs included:

- Minimum width rules
- Minimum spacing rules
- Wide-metal spacing
- Notch rules
- Via sizing
- Multiple contact cuts
- Via enclosure and overlap
- Automatically generated vias
- Minimum-area rules
- Minimum-hole rules
- N-well requirements
- Substrate taps
- Well contacts
- Deep N-well structures
- Full vs. fast DRC styles
- Interactive DRC debugging

A typical debugging cycle was:

```text
Locate DRC Marker
       ↓
Query the Rule
       ↓
Identify Layer + Requirement
       ↓
Measure Geometry
       ↓
Modify Layout
       ↓
Re-run DRC
       ↓
Confirm Error Removal
```

This module helped connect the numerical rules in the PDK with the actual physical structures being fabricated.

---

# 04 — From RTL to GDSII

## Module 4 — OpenLane Digital Physical Design Flow

After working directly with layout and verification, I studied how a digital design moves through the complete **RTL-to-GDSII physical-design flow using OpenLane**.

The flow begins with RTL Verilog and progressively transforms it into a manufacturable physical layout.

```text
RTL Verilog
     │
     ▼
Synthesis
Yosys + ABC
     │
     ▼
Gate-Level Netlist
     │
     ▼
Static Timing Analysis
OpenSTA
     │
     ▼
Floorplanning
     │
     ▼
Placement
     │
     ▼
Clock Tree Synthesis
TritonCTS
     │
     ▼
Global Routing
FastRoute
     │
     ▼
Antenna Handling
     │
     ▼
Detailed Routing
TritonRoute
     │
     ▼
RC Extraction
Magic
     │
     ▼
Post-Layout Timing
     │
     ▼
Physical Verification
DRC + LVS + CVC
     │
     ▼
GDSII
```

### Synthesis

The RTL Verilog design is converted into a gate-level netlist using standard cells from the target technology library.

**Yosys** performs RTL synthesis, while **ABC** performs technology mapping and logic optimization.

### Static Timing Analysis

**OpenSTA** analyzes the synthesized gate-level netlist for timing behavior, including setup and hold constraints.

Before CTS, timing is evaluated using an ideal clock because the physical clock-distribution network has not yet been built.

### Floorplanning

Floorplanning defines the physical organization of the design.

This includes:

- Core area
- Placement rows
- Routing tracks
- I/O placement
- Well-tap insertion
- Decap insertion
- Power Distribution Network generation

### Placement

Standard cells are positioned inside the floorplan while optimizing timing, area, power, and wire length.

The process progresses through global placement, optimization, and detailed placement/legalization.

### Clock Tree Synthesis

**TritonCTS** creates the physical clock-distribution network.

Clock buffers are inserted and balanced to control:

- Clock skew
- Clock latency
- Clock propagation

After CTS, the design contains a real physical clock network rather than an ideal clock.

### Global Routing

**FastRoute** determines approximate routing paths and estimates routing resources.

This stage is also useful for identifying congestion before exact routing is performed.

### Antenna Diode Insertion

Antenna protection is used to reduce plasma-induced charging that can damage transistor gate oxide during fabrication.

OpenLane provides different antenna-handling strategies depending on the design and flow configuration.

### Detailed Routing

**TritonRoute** converts the global routing solution into exact physical wires and vias while satisfying design rules.

The resulting DEF contains the physical placement and routing representation of the design.

### RC Extraction

After routing, **Magic** extracts layout-induced resistance and capacitance.

These parasitics are used for more realistic post-layout timing analysis.

### Physical Verification

The completed physical design is checked using several verification tools:

```text
Magic
→ DRC / antenna checks

KLayout
→ Additional DRC verification

Netgen
→ LVS

CVC
→ Circuit validity checking
```

### GDSII Generation

After verification, the final physical design is streamed out as **GDSII**, the layout format used for fabrication.

### `config.tcl`

The OpenLane flow is controlled through `config.tcl`.

It contains configuration parameters for:

- Synthesis
- Floorplanning
- Placement
- CTS
- Routing
- Timing
- Verification

This module helped connect the individual layout and verification concepts from the earlier modules to a complete automated digital implementation flow.

---

# 05 — When LVS Says the Netlists Do Not Match

## Module 5 — Running LVS & Debugging

The final module focuses on one of the most important questions in physical verification:

> **The layout may be DRC-clean, but does it actually implement the circuit we intended to build?**

Using **Magic, Xschem, SPICE, Verilog, and Netgen**, I progressed from very small LVS examples to hierarchical analog and digital designs.

The learning progression was:

```text
Simple SPICE comparison
          ↓
Subcircuits
          ↓
Blackboxes
          ↓
Primitive SPICE devices
          ↓
Analog hierarchy
          ↓
Layout vs. Verilog
          ↓
Digital macros
          ↓
Large mismatch debugging
          ↓
Property checking
```

### Starting Simple

The first LVS experiments used small SPICE files to understand how Netgen compares:

- Devices
- Nets
- Connectivity
- Pins
- Subcircuits

I intentionally changed individual connections and studied how the errors appeared in `comp.out`.

### Subcircuits & Pin Matching

The next experiments introduced hierarchy and demonstrated that:

```text
Topology matching
```

and:

```text
Top-level pin matching
```

are related but separate checks.

### Blackboxes

I explored how Netgen handles empty subcircuits as blackboxes and why the interface becomes especially important when the internal implementation is unavailable.

This included:

- Blackbox pin matching
- Missing pins
- Proxy pins
- Dummy nets
- Automatic flattening
- The `-blackbox` option

### SPICE Low-Level Components

I then worked with:

- Resistors
- Capacitors
- Diodes

and studied how terminal symmetry affects LVS.

For electrically symmetric circuits, I used Netgen's `permute` configuration to explicitly define legal pin permutations.

### Real Analog LVS

The labs then moved to a real **Power-On Reset (POR)** circuit.

I generated:

```text
Xschem schematic netlist
        ↕
Netgen LVS
        ↕
Magic extracted layout netlist
```

This introduced more realistic issues involving:

- Hierarchy
- PDK devices
- Standard-cell definitions
- Parameterized cells
- Top-level ports
- Metal resistors
- Analog device properties

### Layout vs. Verilog

For digital blocks, I compared:

```text
Magic-extracted SPICE
```

against:

```text
Gate-level structural Verilog
```

This required understanding the difference between:

```text
Behavioral Verilog
```

and:

```text
Structural / synthesized Verilog
```

### Debugging Real LVS Failures

The later labs intentionally introduced realistic errors.

I worked through:

- Missing devices
- Additional devices
- Fill/tap-cell mismatches
- Antenna diode insertion
- Broken power networks
- Missing vias
- Merged top-level ports
- Ground/substrate connectivity
- Hierarchical macro issues

Instead of trying to interpret hundreds of errors at once, I practiced a divide-and-conquer strategy:

```text
Run LVS
   ↓
Check device mismatches
   ↓
Fix obvious structural problems
   ↓
Run LVS again
   ↓
Observe which errors disappear
   ↓
Investigate remaining nets/pins
   ↓
Repeat
```

### Property Checking

The final lab demonstrated that:

> **Topology matching does not automatically mean LVS has passed.**

The layout and schematic could have:

```text
Same devices
Same nets
Same connectivity
Same pins
```

and still fail because:

```text
Device properties differ
```

I debugged mismatches involving:

- MIM capacitor dimensions
- Resistor dimensions
- MOSFET width
- Number of MOSFET fingers

For example:

```text
NFET PCell

Width per finger = 2 µm
Number of fingers = 2

Effective extracted width = 4 µm
```

while the schematic expected:

```text
W = 2 µm
```

Tracing the Netgen property error back to the Magic PCell revealed the actual cause.

---

# 🔍 What This Repository Demonstrates

The central lesson across all five modules is that physical verification is not simply:

```text
Run Tool
   ↓
PASS / FAIL
```

The useful engineering work begins when the result is:

```text
FAIL
```

The debugging process I practiced throughout the repository was:

```text
Observe the Error
        ↓
Understand What the Tool Is Reporting
        ↓
Locate the Responsible Geometry / Device / Net
        ↓
Determine the Root Cause
        ↓
Correct Layout or Schematic
        ↓
Regenerate / Re-extract
        ↓
Run Verification Again
        ↓
Confirm the Error Is Removed
```

This same approach appears throughout the project—from correcting a `0.14 µm` spacing violation to tracing a Netgen property mismatch back to the number of fingers in a MOSFET PCell.

---

# 🧩 Verification Perspective

The different verification stages answer different questions.

| Verification | Question |
|---|---|
| **DRC** | Can this physical geometry be manufactured according to the process rules? |
| **Extraction** | What electrical circuit does this physical geometry represent? |
| **LVS** | Does the extracted circuit match the intended schematic/netlist? |
| **XOR** | Are two physical layout versions geometrically identical? |
| **STA** | Does the implemented design satisfy timing requirements? |
| **CVC** | Is the final circuit electrically valid? |

Together, these checks provide different views of the same physical design.

---

# 📂 Repository Organization

```text
sky130-physical-verification-tapeout/
│
├── Module1_SKY130_Intro/
│   ├── README.md
│   └── Individual lab folders
│
├── Module2_GDS_Extraction_DRC/
│   ├── README.md
│   └── Individual lab folders
│
├── Module3_DRC_Rules/
│   ├── README.md
│   └── Individual DRC lab folders
│
├── Module4_OpenLane/
│   ├── README.md
│   └── OpenLane flow documentation
│
├── Module5_LVS_Debugging/
│   ├── README.md
│   ├── L1_Simple_LVS/
│   ├── L2_LVS_With_Subcircuits/
│   ├── L3_LVS_With_Blackboxes/
│   ├── L4_LVS_With_SPICE_Low_Level_Components/
│   ├── L5_LVS_Small_Analog_Block_POR_Part1/
│   ├── L6_LVS_Small_Analog_Block_POR_Part2/
│   ├── L7_LVS_Layout_vs_Verilog_Standard_Cell/
│   ├── L8_LVS_For_Macros/
│   ├── L9_LVS_Digital_PLL_Part1/
│   ├── L10_LVS_Digital_PLL_Part2/
│   └── L11_LVS_With_Property_Checking/
│
└── README.md
```

Each lab contains its own documentation with:

- Commands used
- Screenshots
- Verification results
- Errors encountered
- Debugging process
- Corrections
- Key observations

---

# 🎯 What I Took Away From This Project

This project helped me connect several topics that are often learned separately.

I started by understanding the technology and creating simple layouts. I then learned why certain geometry is legal or illegal, how that geometry is converted back into an electrical circuit, how a digital RTL design eventually becomes physical geometry, and finally how to prove that the fabricated representation still corresponds to the intended design.

The progression was:

```text
Understand the Technology
          ↓
Create the Layout
          ↓
Check Manufacturability
          ↓
Extract the Circuit
          ↓
Implement RTL Physically
          ↓
Compare Physical vs. Intended Circuit
          ↓
Debug Until Verification Is Clean
```

The biggest takeaway was learning to treat verification reports as **debugging information rather than simply pass/fail results**.

A DRC marker points back to a physical rule.

An LVS mismatch points back to connectivity, hierarchy, pins, devices, or properties.

Understanding how to move from that report back to the actual layout or schematic problem was the most important part of the work documented in this repository.
