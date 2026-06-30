## Soc-Design-and-Planning-VSD

RTL-to-GDSII Physical Design Flow using OpenLane and Sky130 PDK.

## Digital VLSI SoC Design and Planning — RTL to GDSII

This repository contains my notes, lab exercises, screenshots, and learnings from the Digital VLSI SoC Design and Planning Workshop conducted by VSD (VLSI System Design).

---

## Day 1 — Introduction to Open-Source EDA, OpenLANE and Sky130 PDK

### Topics Covered

* Introduction to ASIC Design Flow
* Understanding Chip Components
* RISC-V ISA Basics
* RTL to GDSII Flow
* Open-Source EDA Ecosystem
* Introduction to Sky130 PDK
* OpenLANE ASIC Flow
* Hands-on Synthesis using OpenLANE

---

## Where is a Chip Located & Understanding its Components

### Chip Location

Before diving into the internal architecture, it is essential to understand where the chip resides. A finalized integrated circuit (IC) or chip package is surface-mounted directly onto a **PCB (Printed Circuit Board)**. The PCB contains copper tracks that route electrical power and communication signals between the chip pins and other peripheral components on the system board.

### Inside the Chip

| Component | Description |
|------------|------------|
| **Die** | The actual silicon piece containing the complete circuit. |
| **Core** | Area where standard cells, logic gates, and digital circuitry are placed. |
| **Pads** | Interface between internal circuitry and external pins. |
| **Macros** | Large pre-designed blocks such as SRAMs, PLLs, and processors. |
| **IPs** | Reusable and pre-verified functional blocks. |
| **Foundry** | Semiconductor company responsible for manufacturing the chip. |

---

## RISC-V ISA: From Software to Physical Layout

A software application eventually becomes hardware through several abstraction levels.

```text
C Program
    ↓
Compiler
    ↓
Assembly Code
    ↓
Assembler
    ↓
RISC-V Machine Instructions
    ↓
RTL (Verilog)
    ↓
Netlist
    ↓
Physical Layout
```

### Flow Explanation

- **C Program** → High-level code written by the programmer.
- **Compiler** → Converts C code into assembly instructions.
- **Assembler** → Converts assembly instructions into binary machine code.
- **RISC-V Instructions** → Instructions understood and executed by the processor.
- **RTL (Verilog)** → Hardware description of the processor architecture.
- **Netlist** → Gate-level representation generated after synthesis.
- **Physical Layout** → Final placement of transistors and metal layers on silicon.

---

## Components Required for Open-Source ASIC Design

To successfully execute an automated open-source digital design flow, three fundamental pillars must be present:

### 1. RTL Design

The hardware description code (Verilog) defining the logic and behavior of the target design.

### 2. EDA Tools

Open-source automated software suites used for synthesis, placement, routing, and physical sign-off verification.

* *Examples:* OpenLANE, OpenROAD, Yosys, Magic, Netgen.

### 3. PDK (Process Design Kit)

The collection of files that links software tools to a real manufacturing foundry. It contains:

* Design rules (DRC, LVS)
* Device simulation models
* Standard Cell Libraries (SCL)
* Timing and physical layer information
* *Workshop PDK:* **Sky130 Open-Source PDK**

---

## RTL to GDSII Flow

| Stage | Description |
|---|---|
| **RTL** | Describes functionality of the design using Verilog |
| **Synthesis** | Converts RTL into a gate-level netlist using standard cells |
| **Floorplanning** | Defines chip dimensions, places macros, I/O pins and routing resources |
| **Power Planning** | Creates the PDN to deliver stable power, reduce IR Drop and improve reliability |
| **Placement** | Places standard cells inside core area to reduce wire length and improve timing |
| **CTS** | Builds clock distribution network to minimize skew and balance clock arrival times |
| **Routing** | Creates metal connections between all cells |
| **STA** | Verifies setup, hold and timing requirements |
| **DRC** | Ensures layout follows foundry design rules |
| **LVS** | Verifies physical layout matches the logical netlist |
| **GDSII** | Final layout database sent to foundry for fabrication |

---

## OpenLANE ASIC Flow Tool Reference Stack

The automated pipeline executes the stages listed above using the following open-source tool implementations:

| Stage | Tool |
| :--- | :--- |
| **RTL Synthesis** | Yosys + ABC |
| **STA (Static Timing)** | OpenSTA |
| **DFT (Design for Test)** | Fault |
| **Floorplanning** | OpenROAD |
| **Placement** | OpenROAD |
| **CTS** | TritonCTS |
| **Routing** | TritonRoute |
| **RC Extraction** | OpenRCX |
| **DRC Verification** | Magic |
| **LVS Verification** | Netgen |
| **GDSII Generation** | Magic |

---

### Lab — Running OpenLANE for picorv32a

#### Setting Up and Invoking OpenLANE

```
cd /home/vscode/Desktop/OpenLane
make mount
./flow.tcl -interactive
package require openlane 1.0.2
```

#### Preparing the Design

```
prep -design picorv32a
```

![OpenLANE invoked in interactive mode](images/01_openlane_invoked.png)

#### Running Synthesis

```
run_synthesis
```

![Synthesis completed - part 1](images/02_synthesis_complete1.png)

![Synthesis completed - part 2](images/03_synthesis_complete2.png)

#### Flop Ratio Calculation

After synthesis completes, the flop ratio can be calculated from the synthesis statistics:

```
Flop Ratio = Number of D Flip Flops / Total Number of Cells
           = 1613 / 15762
           = 0.1023
           = 10.23%
```

---

## Day 2 — Floorplanning and Introduction to Library Cells

### Topics Covered

- Floorplanning Fundamentals
- Core and Die Planning
- Aspect Ratio and Utilization Factor
- Macro Placement
- Decoupling Capacitors
- Power Planning
- Pin Placement
- Placement and Optimization
- Standard Cell Design
- Cell Characterization

---

## Floorplanning

Floorplanning is the first physical design stage after synthesis. Its goal is to create an efficient physical structure for the chip before placing standard cells.

### Main Steps

- Define Die Size
- Define Core Size
- Place I/O Pins
- Place Macros
- Build the Power Distribution Network (PDN)
- Create Placement Rows

### Aspect Ratio and Utilization Factor

**Aspect Ratio**

```
Aspect Ratio = Height / Width
```

- Aspect Ratio = 1 → Square Core
- Aspect Ratio > 1 → Tall Core
- Aspect Ratio < 1 → Wide Core

**Utilization Factor**

```
Utilization Factor = Cell Area / Core Area
```

- High Utilization → Congestion and routing difficulties
- Low Utilization → Wasted chip area

A balanced utilization leaves enough space for routing and optimization.

---

## Macro Placement

Macros are large pre-designed blocks such as:

- SRAM
- Processor Cores
- PLLs

These blocks are placed before standard cells and are therefore called **pre-placed cells**.

Good macro placement helps reduce wirelength, congestion and timing issues, resulting in a more efficient layout.

---

## Decoupling Capacitors and Power Planning

During switching, circuits may require a large amount of current instantly. This can cause voltage drops and power noise.

Decoupling capacitors (Decaps) act as local charge reservoirs and supply current during sudden switching activity, helping maintain a stable supply voltage.

The Power Distribution Network (PDN) distributes VDD and VSS throughout the chip using:

- Power Rings
- Vertical Straps
- Horizontal Straps

A well-designed PDN helps reduce:

- IR Drop
- Ground Bounce
- Power Noise

Upper metal layers are preferred because they are wider and thicker, resulting in lower resistance.

---

## Pin Placement

Pins provide connections between the internal circuitry and the external world.

Examples include:

- Input Pins
- Output Pins
- Clock Pins
- Reset Pins

Proper pin placement helps reduce wirelength, improve routing and achieve better timing.

---

## Placement and Optimization

Placement arranges all standard cells inside the placement rows created during floorplanning.

### Placement Stages

**Global Placement**

- Finds an optimized location for each cell.

**Detailed Placement**

- Moves cells to legal locations and aligns them to placement rows.

To improve timing, buffers may be inserted on long signal paths. Buffers regenerate signals and help reduce delay caused by wire resistance and capacitance.

---

## Standard Cell Design

A standard cell is a reusable building block used during synthesis.

Examples:

- Inverter
- NAND Gate
- NOR Gate
- Flip-Flop

A standard cell is created through:

- Circuit Design
- Layout Design
- DRC Verification
- LVS Verification

After verification, the cell becomes part of the Standard Cell Library.

---

## Cell Characterization

After a cell layout is completed, its electrical behavior is measured.

Characterization determines:

- Delay
- Power Consumption
- Noise Characteristics
- Timing Information

The extracted information is stored in a **Liberty (.lib)** file, which is used by synthesis and timing analysis tools throughout the ASIC design flow.

---

## Lab

### Running Floorplan

```bash
run_floorplan
```

![](images/05_Floorplan_run.png)

### Inspecting the Generated DEF File

After floorplanning is completed, the generated DEF file can be inspected using:

```bash
cd runs/<RUN_DIRECTORY>/results/floorplan/
less picorv32a.def
```

![](images/06_floorplan.def.png)

This DEF file contains floorplan information such as:

- Core dimensions
- Placement rows
- Pin locations
- Macro locations
- Floorplan constraints

---

### Viewing the Floorplan in Magic

To visualize the generated floorplan in Magic:

```bash
magic -T /home/vscode/.ciel/ciel/sky130/versions/0fe599b2afb6708d281543108caf8310912f54af/sky130A/libs.tech/magic/sky130A.tech \
lef read ../../tmp/merged.nom.lef \
def read picorv32a.def &
```

### Floorplan Layout

![](images/06_Floorplan.png)

---

### Running Placement

```bash
run_placement
```

![](images/08_Placement_run.png)

### Placement Layout

![](images/09_Placement.png)

![](images/10_Placement_in_detail.png)

---

## Day 3 — Design Library Cell using Magic Layout and ngspice Characterization

### Topics Covered

- Standard Cell Design Flow
- Exploring a Custom Inverter Layout in Magic
- CMOS Fabrication Process Overview
- Layout Extraction (`.ext` → `.spice`)
- SPICE Netlist Editing
- ngspice Transient Simulation
- Cell Characterization (Rise Time, Fall Time, Cell Rise Delay, Cell Fall Delay)
- Magic DRC Lab Exercise

---

## Standard Cell Design — Big Picture

Days 1 and 2 focused on chip-level design. Day 3 zooms into the **cell level** — studying a single standard cell (inverter) from layout all the way to characterization.

```text
Circuit Design
      ↓
Layout Design
      ↓
DRC Verification
      ↓
LVS Verification
      ↓
Extraction
      ↓
SPICE Simulation
      ↓
Characterization
      ↓
.lib Generation
```

---

## Cloning the Custom Inverter Cell

The `vsdstdcelldesign` repository provides a pre-built Sky130 inverter layout for hands-on practice.

```bash
git clone https://github.com/nickson-jose/vsdstdcelldesign
```

Before opening the layout, the Sky130 technology file must be copied into the same directory:

```bash
cp /home/vscode/.ciel/ciel/sky130/versions/0fe599b2afb6708d281543108caf8310912f54af/sky130A/libs.tech/magic/sky130A.tech .
```

---

## Opening the Inverter Layout in Magic

```bash
magic -T sky130A.tech sky130_inv.mag
```

![Custom Inverter Layout opened in Magic](images/stdcelllayout.png)

### Observations from the Layout

- **PMOS** transistor sits in the **N-well** region (upper half of the cell)
- **NMOS** transistor sits in the **P-substrate** region (lower half)
- **VPWR** and **VGND** power rails are visible on Metal1
- Input `A` and Output `Y` ports are accessible on the routing layers
- No DRC violations were present in the original clean layout

---

## Identifying PMOS and NMOS in Magic

By hovering over layers and using the `what` command in the Magic console, the transistors in the layout can be identified.

![PMOS region identified in the inverter layout](images/stdcelllayout2.png)

![NMOS region identified in the inverter layout](images/stdcelllayout3.png)

---

## Layout Extraction

After verifying the layout, extraction is performed inside the Magic Tcl console to convert the physical geometry into an electrical circuit description.

```tcl
extract all
ext2spice cthresh 0 rthresh 0
ext2spice
```

This generates:
- `sky130_inv.ext` — intermediate extraction file with parasitic data
- `sky130_inv.spice` — SPICE netlist ready for simulation

![Extraction commands run in Magic Tcl console](images/stdcelllayout4.png)

![Generated SPICE netlist file](images/created_spicefile.png)

---

## Editing the SPICE Deck

Before simulation, the generated SPICE file was edited to:
- Include the correct PMOS model file (`pshort.lib`) and NMOS model file (`nshort.lib`)
- Correct the model names for both transistors
- Set supply voltage `VDD = 3.3V`
- Add the transient simulation command

![Edited SPICE deck before simulation](images/edited_spicefile.png)

---

## SPICE Simulation using ngspice

```bash
ngspice sky130_inv.spice
```

Inside ngspice, the transient waveform is plotted:

![Generated wave](images/generated_plot.png)


## Cell Characterization

The transient waveform was analyzed to extract key timing parameters:

> VDD = 3.3V → 20% = **0.66V** | 50% = **1.65V** | 80% = **2.64V**

### Rise Transition
Time taken by the output to rise from **20% to 80%** of VDD.
* **Value:** `0.064 ns`

![Zoomed waveform for Rise Transition measurement](images/rise_time20.png)
![Zoomed waveform for Rise Transition measurement](images/rise_time80.png)
![Zoomed waveform for Fall Transition measurement](images/risetime_values.png)

### Fall Transition
Time taken by the output to fall from **80% to 20%** of VDD.
* **Value:** Obtained similarly by measuring the falling edge times.

![Zoomed waveform for Fall Transition measurement](images/fall_time20.png)
![Zoomed waveform for Fall Transition measurement](images/fall_time80.png)
![Zoomed waveform for Fall Transition measurement](images/falltime_values.png)

### Cell Rise Delay
Time difference between **50% of input** (falling) and **50% of output** (rising).

![Zoomed waveform for Cell Rise Delay measurement](images/propdelay_graph.png)
![Zoomed waveform for Cell Rise Delay measurement](images/propdelay_values.png)

These values feed into Liberty (`.lib`) files used by synthesis and STA tools throughout the ASIC flow.

---

## Lab — Magic DRC Exercise

Using the Magic layout tool and the Sky130 technology rule file (`sky130A.tech`) to find and fix physical geometry violations.

### 1. Fixing Poly Spacing Rules (`poly.9`)
* Searched the `.tech` file for the `drc` section and located the `poly.9` rule definition.
* Inspected the `/aliases` block and modified the missing spacing parameters to clear the layout errors.
* Reloaded the updated technology rule deck and forced a fresh check inside Magic:
  ```tcl
  .techload sky130A.tech
  drc check
