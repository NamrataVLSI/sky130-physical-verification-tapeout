# SKY130 Physical Verification Labs

This repository documents my hands-on learning and lab work in **IC physical verification using the SkyWater SKY130 PDK and open-source EDA tools**.

The project covers the complete physical verification flow, starting from understanding the SKY130 process and layout environment, progressing through **DRC**, and finally performing **LVS and debugging layout-vs-schematic mismatches**.

---

## Tools Used

- **SkyWater SKY130 PDK**
- **Magic VLSI** — Layout, DRC, and extraction
- **Xschem** — Schematic capture and SPICE netlist generation
- **Netgen** — Layout Versus Schematic (LVS)
- **Ngspice** — SPICE simulation
- **Linux / TCL / Shell**

---

# Modules

## Module 1 — SKY130 & Physical Verification Fundamentals

Introduction to the **SKY130 technology, PDK structure, fabrication layers, layout concepts, and physical verification flow**.

**Key topics:** SKY130 process, layout layers, CIF/GDS concepts, design rules, and physical verification fundamentals.

---

## Module 2 — Magic Layout Fundamentals

Hands-on introduction to **Magic VLSI** and the basic layout editing environment.

**Key topics:** layout navigation, layer selection, geometry creation/editing, box operations, cell hierarchy, and Magic commands.

---

## Module 3 — Design Rule Checking (DRC)

Practical debugging of different **SKY130 design-rule violations** using Magic.

**Key topics:**

- Width and spacing rules
- Via/contact rules
- Minimum-area and minimum-hole rules
- Wells and substrate taps
- Deep N-well rules
- DRC error inspection and correction

---

## Module 4 — LVS Fundamentals

Introduction to the concepts behind **Layout Versus Schematic verification** and how circuit netlists are compared.

**Key topics:** extracted netlists, hierarchical comparison, black boxes, pin matching, device matching, property checking, series/parallel device handling, and interpreting Netgen results.

---

## Module 5 — Running LVS & Debugging

Hands-on LVS labs using **Magic, Xschem, and Netgen** to compare extracted layout netlists against schematic or Verilog netlists.

**Key topics:**

- Basic Netgen LVS
- LVS with subcircuits
- Black-box handling
- SPICE low-level components
- Analog block LVS
- Layout vs. gate-level Verilog
- Hierarchical and macro-based LVS
- Debugging net/device/pin mismatches
- Property mismatch debugging
- MOSFET, resistor, and capacitor property checking

---

# Overall Verification Flow

```text
        Schematic / Verilog
                │
                ▼
         Reference Netlist
                │
                │
Layout ──► DRC ──► Extraction
                │
                ▼
         Extracted Netlist
                │
                ▼
             Netgen
                │
                ▼
              LVS
                │
       ┌────────┴────────┐
       ▼                 ▼
     PASS             MISMATCH
                         │
                         ▼
                      Debug
                         │
                         ▼
                    Run Again
