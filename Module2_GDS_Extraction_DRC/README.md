# Module 2 — GDS, Extraction, DRC, LVS and XOR Verification

## Overview

This module focused on the core physical-verification flow using the **SKY130 PDK** and **Magic**.

I worked through the complete process of reading GDS data, inspecting port metadata, understanding abstract views, extracting SPICE netlists, running DRC and LVS, and finally comparing layout geometry using XOR.

The module covered both interactive and batch verification workflows.

---

## Module Flow

```text
GDS Read
   ↓
Port and Metadata Inspection
   ↓
LEF / Abstract Views
   ↓
Layout Extraction
   ↓
Parasitic RC Extraction
   ↓
DRC Verification
   ↓
LVS Comparison
   ↓
XOR Layout Comparison
```

---

## Labs

### L1 — GDS Read

Explored how Magic reads SKY130 standard-cell GDS data.

Topics covered:

- GDS import
- CIF/GDS input styles
- vendor input style
- Cell Manager
- standard-cell loading
- duplicate-cell handling

Folder:

```text
L1_GDS_Read/
```

---

### L2 — Ports

Studied how port information is represented in layout files and how metadata differs between GDS, LEF, and SPICE.

Topics covered:

- `port index`
- port names
- port class
- port use
- limitations of GDS metadata
- LEF annotation
- SPICE-based port ordering

Folder:

```text
L2_Ports/
```

---

### L3 — Abstract Views

Worked with LEF abstract views and explored how Magic handles library cells and GDS references.

Topics covered:

- reading LEF data
- abstract cell views
- library search paths
- GDS metadata
- read-only vendor cells
- writing and re-reading GDS
- hierarchy preservation

Folder:

```text
L3_Abstract_Views/
```

---

### L4 — Basic Extraction

Extracted electrical information from physical layout and generated SPICE netlists.

Topics covered:

- `extract all`
- `.ext` files
- LVS-oriented SPICE extraction
- parasitic capacitance extraction
- capacitance thresholding
- resistance extraction
- RC-aware post-layout netlists

Folder:

```text
L4_Basic_Extraction/
```

---

### L5 — Setup for DRC

Ran both batch and interactive Design Rule Checking.

Topics covered:

- OpenPDK batch DRC
- `drc(fast)` vs `drc(full)`
- `drc check`
- `drc why`
- `drc find`
- well and substrate tap rules
- hierarchical DRC
- tap/end-cap cells
- achieving `DRC=0`

Folder:

```text
L5_Setup_DRC/
```

---

### L6 — Setup for LVS

Set up a basic LVS comparison using Magic and Netgen.

Topics covered:

- LVS-oriented layout extraction
- layout vs. reference SPICE comparison
- Netgen batch mode
- `"file cell"` input format
- device and net comparison
- LVS mismatch observation and debugging setup

Folder:

```text
L6_Setup_LVS/
```

---

### L7 — Setup for XOR

Used Magic's XOR operation to compare two layout versions.

Topics covered:

- creating local editable copies
- `flatten -nolabels`
- `xor -nolabels`
- detecting local geometry changes
- detecting shifted standard-cell instances
- comparing layout revisions

Folder:

```text
L7_Setup_XOR/
```

---

## Key Tools

- Magic
- Netgen
- SKY130 PDK
- SPICE
- LEF
- GDS

---

## Key Takeaways

By the end of this module, I was able to:

- Read SKY130 GDS libraries into Magic
- Inspect layout ports and metadata
- Understand the role of GDS, LEF, and SPICE files
- Work with abstract views and vendor library cells
- Extract layout connectivity into SPICE
- Generate parasitic RC netlists
- Run batch and interactive DRC
- Understand hierarchical DRC behavior
- Set up an LVS comparison using Netgen
- Use XOR to locate geometry differences between layout revisions

---

## Verification Concepts Covered

| Check | Purpose |
|---|---|
| DRC | Verify layout geometry against process rules |
| LVS | Verify electrical equivalence between layout and reference circuit |
| XOR | Verify geometric equivalence between two layouts |
| Extraction | Convert physical layout into an electrical representation |

---

## Module Result

```text
GDS Read                    ✓
Port Inspection             ✓
LEF / Abstract Views        ✓
Basic Extraction            ✓
Parasitic RC Extraction     ✓
Batch DRC                   ✓
Interactive DRC             ✓
Hierarchical DRC            ✓
LVS Setup                   ✓
XOR Layout Comparison       ✓
```

This module established the complete foundation for **SKY130 physical verification**, from reading layout databases through extraction, DRC, LVS, and geometric comparison.
