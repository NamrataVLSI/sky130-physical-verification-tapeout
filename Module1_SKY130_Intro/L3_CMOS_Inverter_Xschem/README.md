# L3 — CMOS Inverter Schematic using Xschem

## Objective

Create a CMOS inverter schematic using SKY130 devices in Xschem and configure the NMOS and PMOS device parameters for subsequent simulation and physical verification.

---

## 1. Creating a New Schematic

Xschem was launched from the project `xschem` directory.

A new schematic was created using:

```text
File → New Schematic
```

The **Insert** key was used to open the symbol selection window.

---

## 2. Selecting SKY130 Devices

The SKY130 primitive device library was accessed from:

```text
/usr/share/pdk/sky130A/libs.tech/xschem/sky130_fd_pr
```

The following transistor symbols were selected:

- `nfet_01v8.sym` — NMOS transistor
- `pfet3_01v8.sym` — PMOS transistor

![SKY130 Device Library](images/01_symbol_library.png)

![SKY130 MOSFET Devices](images/02_sky130_devices.png)

---

## 3. Adding Input, Output, and Supply Pins

From the Xschem device library, the following symbols were used:

```text
ipin.sym
opin.sym
iopin.sym
```

The pins were renamed as:

- `IN` — inverter input
- `OUT` — inverter output
- `VDD` — positive supply
- `VSS` — ground supply

The **W** key was used to wire the devices.

The PMOS and NMOS gates were connected together to form `IN`, while their drains were connected together to form `OUT`.

The PMOS source was connected to `VDD` and the NMOS source was connected to `VSS`.

![CMOS Inverter Wiring](images/03_inverter_wiring.png)

---

## 4. NMOS Configuration

The NMOS properties were edited and configured as:

```text
name=M1
L=0.18
W=4.5
nf=3
mult=1
model=nfet_01v8
```

![NMOS Properties](images/04_nmos_properties.png)

---

## 5. PMOS Configuration

The PMOS properties were configured as:

```text
name=M2
L=0.18
W=3
nf=3
mult=1
body=VDD
```

![PMOS Properties](images/05_pmos_properties.png)

---

## 6. Final CMOS Inverter Schematic

The completed inverter consists of a PMOS pull-up device and an NMOS pull-down device.

When `IN` is LOW, the PMOS turns ON and pulls `OUT` toward `VDD`.

When `IN` is HIGH, the NMOS turns ON and pulls `OUT` toward `VSS`.

![Final CMOS Inverter](images/06_final_inverter.png)

---

## 7. Saving the Schematic

The completed schematic was saved as:

```text
inverter.sch
```

using:

```text
File → Save As → inverter.sch
```

## Result

Successfully created and configured a SKY130 CMOS inverter schematic in Xschem with `IN`, `OUT`, `VDD`, and `VSS` connections. The schematic was saved as `inverter.sch` for subsequent simulation and physical layout steps.
