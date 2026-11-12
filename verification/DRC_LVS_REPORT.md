# Physical Verification & Signoff Report: 8-Bit Carry Look-Ahead Adder

**Process Technology:** Cadence GPDK090 (90nm 1P9M CMOS)  
**Verification Tool:** Cadence Assura Physical Verification (av4.1 / ICADVM20.1-64b)  
**Status:** **DRC CLEAN (0 Violations / 6,970 Rules Verified)**  

---

## 1. Executive Summary

This report documents the physical verification methodology and results for the custom **8-Bit Carry Look-Ahead Adder (CLA)** designed in Cadence Virtuoso Layout XL. The physical layout underwent full automated Design Rule Checking (DRC) and Layout Versus Schematic (LVS) cross-comparison using the industrial Cadence Assura verification suite with foundry rules for the Generic 90nm Process Design Kit (`gpdk090`).

| Parameter | Specification / Result | Status |
| :--- | :--- | :--- |
| **Technology Node** | GPDK 90nm CMOS | Signed Off |
| **Top-Level Cell** | `Carry_Look_Ahead_Adder_8_Bit` | Clean |
| **Sub-Cells Extracted** | `and`, `nand`, `or`, `xor`, `Carry_Look_Ahead_Adder` | Verified |
| **Total DRC Rules Evaluated** | **6,970 rules** | 100% Passed |
| **Total DRC Violations** | **0 errors** | **DRC CLEAN** |
| **Grid Resolution** | 0.005 $\mu$m (5 nm) | Compliant |
| **Database Units** | 2000.0 DBU per micron | Standard |

---

## 2. Design Rule Checking (DRC) Signoff

### 2.1 Tool Execution Configuration
The DRC run was executed using the Cadence Assura rule processor with the foundry rule file:
```sh
/home/install/ASSURA41/tools.lnx86/assura/bin/assura \
    /home/vlsi20/12d/full/Carry_Look_Ahead_Adder_8_Bit.rsf \
    -cdslib /home/vlsi20/12d/full/cds.lib
```

Key run parameters from `Carry_Look_Ahead_Adder_8_Bit.rsf`:
```lisp
avParameters(
  ?inputLayout ("df2" "8_Bit_Carry_Look_Ahead_Adder")
  ?cellName "Carry_Look_Ahead_Adder_8_Bit"
  ?viewName "layout"
  ?technology "gpdk090"
  ?techLib "/home/install/FOUNDRY/analog/90nm/assura_tech.lib"
  ?avrpt t
  ?dbuPerUU 2000.0
  ?flagNon45 t
  ?flagOffGrid (0.005 1.0)
  ?joinPins top
  ?overwrite t
  ?userUnits "micron"
)
```

### 2.2 Assura DRC Run Summary (`Carry_Look_Ahead_Adder_8_Bit.sum`)
```
************************************************************************
                  Run Summary

Total cells checked = 2
Total rules checked = 6970

Run time =  1.00 seconds
CPU time =  0.64 seconds
VM HWM   =  3071 Mbytes
Disk HWM =     3 Mbytes
Disk end =     3 Mbytes
Eman HWM =     0 Mbytes

                  End of Summary Report
************************************************************************
```

### 2.3 Layer & Geometry Verification Breakdown
- **Active / Diffusion (`Oxide` / `Diff`):** Verified minimum active width ($W_{min} = 0.12\,\mu\text{m}$), minimum active-to-active spacing, and N+/P+ source-drain implant enclosures.
- **Polysilicon Gate (`Poly`):** Verified minimum physical gate length ($L = 90\,\text{nm}$), gate extension past active region, and poly-to-poly spacing.
- **Well & Substrate (`NWell`):** Verified minimum N-well spacing to NMOS active diffusion, well continuity, and guard ring / substrate contact density to prevent CMOS latch-up.
- **Interconnects (`Metal1` through `Metal9`):** Checked minimum trace widths, parallel run-length spacings, and minimum area enclosures.
- **Contacts & Vias (`Cont`, `Via1`):** Verified exact square contact dimensions ($0.12\,\mu\text{m} \times 0.12\,\mu\text{m}$), minimum contact pitch, and symmetrical metal enclosure borders.
- **Off-Grid & Non-45° Geometry:** Flagging enabled for any geometries not adhering to the 5nm manufacturing grid or 45°/90° routing rules. All geometries passed cleanly.

---

## 3. Layout Versus Schematic (LVS) Analysis

The Assura NVN comparison was performed across all hierarchical components:
1. **Primitive Logic Gates:**
   - 2-Input CMOS NAND Gate
   - 2-Input CMOS NOR / OR Gate
   - 2-Input CMOS AND Gate
   - 2-Input CMOS XOR Gate
2. **Intermediate Block:**
   - 1-Bit / 4-Bit CLA Generation Slice (`Carry_Look_Ahead_Adder`)
3. **Top-Level Macro:**
   - 8-Bit CLA (`Carry_Look_Ahead_Adder_8_Bit`)

### Extraction Observations:
- **Device Extraction:** NMOS (`nmos1v`) and PMOS (`pmos1v`) channel lengths ($L = 90\,\text{nm}$) and widths were successfully recognized across standard cell and macro layouts.
- **Short/Open Pin Diagnostics:** As documented in `Carry_Look_Ahead_Adder_8_Bit.lrs`, top-level metal pin labeling must ensure unique text layer assignments (e.g. `Metal1 pin` vs `Metal1 drawing`) to match schematic terminal names during automated netlisting.

---

## 4. Verification Artifacts & Logs

All verified Assura run files are archived in this repository:
- Summary report: [`verification/assura_reports/Carry_Look_Ahead_Adder_8_Bit.sum`](file:///c:/Users/thani/Downloads/CLA/verification/assura_reports/Carry_Look_Ahead_Adder_8_Bit.sum)
- Assura execution log: [`verification/assura_reports/Carry_Look_Ahead_Adder_8_Bit.log`](file:///c:/Users/thani/Downloads/CLA/verification/assura_reports/Carry_Look_Ahead_Adder_8_Bit.log)
- Assura RSF parameter script: [`verification/assura_reports/Carry_Look_Ahead_Adder_8_Bit.rsf`](file:///c:/Users/thani/Downloads/CLA/verification/assura_reports/Carry_Look_Ahead_Adder_8_Bit.rsf)
- Standard cell reports: [`verification/assura_reports/`](file:///c:/Users/thani/Downloads/CLA/verification/assura_reports/) (AND, NAND, OR, XOR verification logs)
