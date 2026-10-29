## 1. Quick Reference & Commands

```bash
# Intial setup
tcsh

# Setup environment (paths, Cadence Innovus, Synopsys DC)
source envsetup

# Full Flow (Synthesis + APR in batch mode)
SJapr flow

# Individual Stages (Batch)
SJapr syn        # Synopsys Design Compiler
SJapr apr        # Cadence Innovus

# Interactive Debugging Mode (-i)
SJapr -i syn
SJapr -i apr
```

### Interactive Mode Helper Commands (within DC or Innovus shell)
* `sj_list_steps`     : Displays remaining queued steps.
* `sj_next_step`      : Runs the next step in sequence.
* `sj_skip_step`      : Skips the current step.
* `sj_run_remaining`  : Runs all remaining steps until failure or end.

### Reloading a Saved Design in Innovus
```tcl
restoreDesign outputs/apr/DBS/<design_name>.enc.dat <design_name>
```

---

## 2. Directory Layout & Auto-Archiving

* **Auto-Archive (`rename_rundir.tcsh`)**: Before any run, existing `outputs/synthesis` or `outputs/apr` folders are automatically backed up as `<rundir>_<MMDD>_<HHMM>` based on previous log timestamps.
* **Output Structure**:
  * `outputs/<stage>/DBS/` : Netlists (`.v`, `.phy.v`), `.def`, `.gds`, `.lef`, `.spef`, `.sdf`, `.enc`
  * `outputs/<stage>/RPT/` : Timing (`timing_max.rpt`), area, power, DRC (`.geom.rpt`), LVS (`.conn.rpt`), CTS
  * `outputs/<stage>/LOG/` : Per-step logs (`<step_name>.log`) and overall runtime log (`flow_summary.log`)

---

## 3. Technology & Target Parameters

* **Process**: SkyWater Sky130 (130 nm CMOS)
* **Standard Cell Library**: `sky130_scl_9T` (9-track, standard, HS, LP)
* **Target Clock Period**: `2.5 ns` (400 MHz)
* **Supply Rails**:
  * Core Power: `vccd1` (1.8 V)
  * Ground: `vssd1` (0.0 V)
* **Macros**:
  * Ring Oscillator / Clock Gen: `clockComplex`
  * Embedded Memory: `CF_SRAM_1024x32`

---

## 4. Synthesis Flow Map (Synopsys Design Compiler)

| Step | Script | Action / Purpose | Key Outputs / Artifacts |
| :--- | :--- | :--- | :--- |
| **1. Init** | `step_init_synthesis.tcl` | Suppresses cosmetic tool warnings (e.g. `UID-401`). | Clean startup log |
| **2. Libraries** | `step_define_library.tcl` | Loads Sky130 `.db` libraries (typical, fast, slow) and macro `.db`s. Sets min/max library pairing for setup/hold. Sets `set_dont_use` on scan FFs. | `target_library`, `link_library` |
| **3. Read RTL** | `step_read_rtl.tcl` | Runs `get_rtl.py` to auto-resolve dependencies. Runs `analyze` (linting) and `elaborate` (GTECH translation). Runs `check_design`. | `file_list.f`, `$DESIGN.svf`, `report.check` |
| **4. Environment** | `step_environment.tcl` | Sets On-Chip Variation (OCV). Applies driving cell (`BUFX2`) to inputs, infinite drive on clock ports, and pin loads on outputs. | Operating condition settings |
| **5. Constraints** | `step_constraints.tcl` | Creates physical clocks (`coreClk`, `spiClk`, `pixelClk`) at `CLOCK_PERIOD = 2.5ns`. Sets `set_fix_hold`. | Clocks and timing constraints |
| **6. Compile** | `step_compile.tcl` | Protects feedthroughs (`set_fix_multiple_port_nets`). Runs `compile_ultra -no_autoungroup -no_boundary_optimization` and maps to Sky130 gates. Runs area optimization. | Gate-level netlist in memory |
| **7. DFT** | `step_dft.tcl` | *(Optional)* Configures scan chains, runs DRC, and inserts scan chains. | Scan insertion reports |
| **8. Reporting** | `step_report.tcl` | Reports setup/hold timing, constraints, cell area, NAND2-equivalent gate counts, and dynamic/static power. | `timing_max.rpt`, `area.rpt`, `power.rpt` |
| **9. Outputs** | `step_write_outputs.tcl` | Fixes net names (`define_name_rules`). Exports handoff files. | `$DESIGN.gate.v`, `$DESIGN.sdc`, `$DESIGN.ddc`, `$DESIGN.sdf` |

---

## 5. Automated Place & Route Flow Map (Cadence Innovus)

| Step | Script | Action / Purpose | Key Outputs / Artifacts |
| :--- | :--- | :--- | :--- |
| **1. Init APR** | `step_init_apr.tcl`<br>`step_init_design.tcl`<br>`step_corners.tcl` | Allocates 8 CPU threads. Loads tech LEF, standard cell LEFs, macro LEFs. Reads SDC and gate netlist. Configures MMMC views (typical RC corner with Quantus `qrcTechFile`). | Initialized Innovus database |
| **2. Floorplan** | `step_floorplan.tcl` | Ingests `openframe_project_wrapper.def`. Places SRAM macros and clock complex at fixed locations with halos. Builds core/block power rings and M4/M5 power stripes. Connects supplies via `sroute`. | Complete PDN and macro floorplan |
| **3. Placement** | `step_place_std_cells.tcl`<br>`step_place_ties.tcl` | Runs timing-driven global and detailed placement (`place_design`). Resolves congestion (`congRepair`). Inserts `TIEHI` / `TIELO` cells to protect gate oxide. | Placed design, `checkPlace` report |
| **4. Pre-CTS Opt** | `step_pre_cts_opt.tcl` | High effort pre-CTS optimization (`optDesign -preCTS`). Fixes max transition and max capacitance violations. | Pre-CTS timing reports |
| **5. CTS** | `step_cts.tcl` | Runs Cadence CCOpt. Builds balanced clock trees using `CLKBUFX*` and `CLKINVX*`. Target skew $\le 150$ ps, max trans $\le 150$ ps. | `clock_opt_design`, CTS reports |
| **6. Post-CTS Opt** | `step_post_cts_opt.tcl` | Fixes hold violations introduced by clock skew (`optDesign -postCTS -hold`). Saves intermediate checkpoint. | `$DESIGN.postCts.dat` |
| **7. NanoRoute** | `step_route.tcl` | Detailed routing with NanoRoute (`routeDesign`). Fixes antenna violations (`drouteFixAntenna`). Optimizes wire length and runs post-route setup/hold opt. | Fully routed design |
| **8. Fillers** | `step_fill.tcl` | Adds standard filler cells (`FILL1`–`FILL8`) and decoupling capacitors (`FILL_DECAP16`) for well continuity and supply noise reduction. | DRC-clean active cell array |
| **9. Physical Verification** | `step_physical_verify.tcl` | Runs geometric DRC check (`verifyGeometry`) and logical-to-physical open/short connectivity verification (`verifyConnectivity`). | `$DESIGN.geom.rpt`, `$DESIGN.conn.rpt` |
| **10. Signoff Export** | `step_extract.tcl` | Extracts RC parasitics (`extractRC`), writes SPEF, SDF, abstract LEF, layout DEF, liberty timing model, GDSII mask, and logical/physical netlists. | `$DESIGN.gds`, `$DESIGN.spef`, `$DESIGN.def`, `$DESIGN.enc` |

---

## 6. Key Design Tips & Troubleshooting

1. **Gate Oxide Protection**: Always ensure `step_place_ties.tcl` runs successfully. Tying logic inputs directly to `VDD` or `VSS` power rails instead of `TIEHI`/`TIELO` cells causes antenna/ESD DRC failures and risks gate oxide breakdown during fabrication.
2. **Clock Tree Isolation**: In `step_environment.tcl`, clock ports are assigned `set_drive 0` to prevent Design Compiler from inserting standard buffers during logic synthesis. Clock tree buffers must only be inserted during backend CTS (`step_cts.tcl`) by Innovus CCOpt.
3. **Macro Halos**: Keepout halos (21 µm around SRAMs, 18 µm around clock complex) prevent standard cells from being placed too close to macro boundaries, preventing local routing congestion around macro pins.
4. **Physical Netlist vs. Gate Netlist**:
   * `$DESIGN.gate.v`: Logical netlist without power pins or physical filler cells (used for functional simulation and formal verification).
   * `$DESIGN.phy.v`: Physical netlist containing explicit `vccd1` and `vssd1` power/ground pin connections (used for LVS and post-layout power analysis).