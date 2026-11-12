# Cadence Virtuoso IC6.1.8 & Assura Reproduction Guide

This guide describes how to import, inspect, simulate, and physically verify this **8-Bit and 16-Bit Carry Look-Ahead Adder (CLA)** project using **Cadence Virtuoso (IC6.1.8 / ICADVM20.1)** and **Assura Physical Verification**.

---

## 1. Prerequisites & Environment Setup

Ensure the following EDA software tools and foundry PDK are installed on your Linux workstation:
- **Cadence Virtuoso Design Framework II (DFII):** IC6.1.8 or ICADVM20.1-64b
- **Spectre Circuit Simulator:** Version 20.1 or newer
- **Assura Physical Verification Suite:** Version av4.1
- **Foundry PDK:** Cadence GPDK090 (Generic Process Design Kit, 90nm CMOS)

### Setting Shell Environment Variables (Bash/Csh)
```bash
# Set Cadence installation base paths
export CDS_HOME=/cadence/IC618
export PATH=$CDS_HOME/tools/bin:$CDS_HOME/tools/dfII/bin:$PATH
export CDS_Netlisting_Mode=Analog

# Assura environment
export ASSURAHOME=/cadence/ASSURA41
export PATH=$ASSURAHOME/tools/assura/bin:$PATH

# Foundry PDK path
export GPDK090_PATH=/foundry/analog/90nm
```

---

## 2. Library Setup & cds.lib Configuration

Clone the repository to your local workspace:
```bash
git clone https://github.com/<your-username>/8-Bit-Carry-Look-Ahead-Adder-Cadence-90nm.git
cd 8-Bit-Carry-Look-Ahead-Adder-Cadence-90nm/cadence
```

Open `cds.lib` and configure the technology mappings to match your local installation:
```lisp
# Include standard Cadence libraries
SOFTINCLUDE $CDS_HOME/share/cdssetup/cds.lib

# Include Generic 90nm Process Design Kit
DEFINE gpdk090 /foundry/analog/90nm/gpdk090
DEFINE avTech $ASSURAHOME/tools.lnx86/assura/etc/avtech/avTech

# Project Design Libraries (Relative Paths)
DEFINE 8_Bit_Carry_Look_Ahead_Adder ./8_Bit_Carry_Look_Ahead_Adder
DEFINE 16_Bit_Carry_Look_Ahead_Adder ./16_Bit_Carry_Look_Ahead_Adder
```

Launch Cadence Virtuoso:
```bash
virtuoso &
```

---

## 3. Opening Schematics & Symbols

1. In the **Cadence Library Manager**, navigate to library `8_Bit_Carry_Look_Ahead_Adder`.
2. Inspect the primitive logic gates:
   - `nand`: Open `schematic` and `symbol`
   - `and`: Open `schematic` and `symbol`
   - `or`: Open `schematic` and `symbol`
   - `xor`: Open `schematic` and `symbol`
3. Inspect the bit slice:
   - `Carry_Look_Ahead_Adder`: Open `schematic` and `symbol`
4. Inspect the top-level adder:
   - `Carry_Look_Ahead_Adder_8_Bit`: Open `schematic` and `symbol`
5. Inspect the testbench:
   - `Carry_Look_Ahead_Adder_8_Bit_sim`: Open `schematic`

---

## 4. Running Transient Simulation in ADE L / Spectre

1. From the testbench schematic (`Carry_Look_Ahead_Adder_8_Bit_sim`), select **Launch -> ADE L**.
2. In ADE L:
   - **Simulator:** Choose `spectre`.
   - **Model Libraries:** Confirm `gpdk090.scs` (`stat`, `tt`, `mc`, or `ff` corners) is linked.
   - **Analyses:**
     - Type: `tran`
     - Stop Time: `100n`
     - Accuracy: `moderate` or `conservative`
   - **Outputs -> To Be Plotted -> Select On Design:**
     - Select `/Cout`
     - Select outputs `/S0` through `/S7`
     - Select inputs `/net1` through `/net19`
3. Click the **Green Play Button (Netlist and Run)**.
4. The **ViVA (Virtuoso Visualization and Analysis)** waveform window opens displaying the transient transitions.
5. You can export transient CSV data by choosing **File -> Save As -> CSV** in ViVA (matches `simulations/8_Bit_CLA_CSA.csv`).

---

## 5. Physical Layout & DRC/LVS Verification

### 5.1 Viewing the Custom Layout
1. In Library Manager, open `Carry_Look_Ahead_Adder_8_Bit -> layout`.
2. Press `Shift + F` to unhide all hierarchical layout levels.
3. Observe:
   - Power and Ground rails (`VDD` and `VSS`) routed on `Metal1` and `Metal2`.
   - Poly gates with contacts to Metal1.
   - N-Well regions enclosing PMOS arrays.
   - Input/output boundary pins.

### 5.2 Running Assura DRC
1. In the layout editor, choose **Assura -> Run DRC...**.
2. Configure the Run parameters:
   - **Technology:** `gpdk090`
   - **Rule Set:** `default`
   - **Tech Library:** Point to `/foundry/analog/90nm/assura_tech.lib`
3. Click **OK** to run.
4. Assura completes with **0 DRC violations** (`.sum` report matches `verification/assura_reports/Carry_Look_Ahead_Adder_8_Bit.sum`).

### 5.3 Running Assura LVS
1. In the layout editor, choose **Assura -> Run LVS...**.
2. Select:
   - **Schematic Library:** `8_Bit_Carry_Look_Ahead_Adder`
   - **Schematic Cell:** `Carry_Look_Ahead_Adder_8_Bit`
   - **Schematic View:** `schematic`
3. Enable device extraction and net comparison.
4. Click **OK** to run.
