# LibreLane & Magic Comprehensive Cheatsheet

A complete reference guide for physical design flows using LibreLane, including all steps, configuration variables, dependencies, and code usage examples.

## Table of Contents
- [1. Quickstart Script Template](#1-quickstart-script-template)
- [2. Command Line Execution](#2-command-line-execution)
- [3. Built-In Step Reference](#3-built-in-step-reference)
  - [Checker Steps](#checker-steps)
  - [KLayout Steps](#klayout-steps)
  - [Magic Steps](#magic-steps)
  - [Misc Steps](#misc-steps)
  - [Netgen Steps](#netgen-steps)
  - [Odb Steps](#odb-steps)
  - [OpenROAD Steps](#openroad-steps)
  - [Verilator Steps](#verilator-steps)
  - [Yosys Steps](#yosys-steps)
- [4. Configuration Variables Reference](#4-configuration-variables-reference)
  - [Universal Flow Variables](#universal-flow-variables)
  - [Checker Step-Specific Variables](#checker-step-specific-variables)
  - [KLayout Step-Specific Variables](#klayout-step-specific-variables)
  - [Magic Step-Specific Variables](#magic-step-specific-variables)
  - [Misc Step-Specific Variables](#misc-step-specific-variables)
  - [Netgen Step-Specific Variables](#netgen-step-specific-variables)
  - [Odb Step-Specific Variables](#odb-step-specific-variables)
  - [OpenROAD Step-Specific Variables](#openroad-step-specific-variables)
  - [Verilator Step-Specific Variables](#verilator-step-specific-variables)
  - [Yosys Step-Specific Variables](#yosys-step-specific-variables)
- [5. Visualizing in Magic (noVNC Desktop)](#5-visualizing-in-magic-novnc-desktop)

## 1. Quickstart Script Template
Copy and paste this template into a `.py` file (e.g., `custom_run.py`) to build your own custom flow:

```python
from librelane.flows import SequentialFlow
from librelane.steps import Yosys, OpenROAD, Magic, Odb, Netgen, KLayout

class CustomRun(SequentialFlow):
    # Group steps in your desired order of execution
    Steps = [
        Yosys.Synthesis,
        OpenROAD.Floorplan,
        Odb.CustomIOPlacement,
        OpenROAD.GlobalPlacement,
        OpenROAD.DetailedPlacement,
        OpenROAD.GlobalRouting,
        OpenROAD.DetailedRouting,
        OpenROAD.RCX,
        Magic.StreamOut,
        Magic.WriteLEF,
        OpenROAD.STAPostPNR,
        Magic.DRC
    ]

flow = CustomRun(
    config={
        "PDK": "sky130A",
        "DESIGN_NAME": "pm32",
        "VERILOG_FILES": ["./pm32.v", "./spm.v"],
        "CLOCK_PORT": "clk",
        "CLOCK_PERIOD": 25.0,
        "IO_PIN_ORDER_CFG": "./pin_order.cfg"
    },
    design_dir="."
)

flow.start()
```

## 2. Command Line Execution

* **Enter the Tool Environment (noVNC Terminal)**:
  ```bash
  librelane
  ```
  *(Drops you into the container shell containing all EDA binaries).*

* **Run a Custom Python Flow Script (inside container)**:
  ```bash
  python3 custom_run.py
  ```

* **Run a Standard Flow Config via CLI (inside container)**:
  ```bash
  python3 -m librelane ./designs/pm32/config.json
  ```

## 3. Built-In Step Reference
Below is the complete list of all registered step classes in LibreLane, grouped by tool/module namespace.

### Checker Steps

#### `Checker.DisconnectedPins`
Raises an immediate error if critical disconnected pins (metric: ``design__critical_disconnected_pin__count``) are >= 0. Doesn't raise an error depending on error_on_var if defined.

* **Inputs**: None
* **Outputs**: None
* **Precursor Steps (Dependencies)**:
None (typically runs at the start of the flow, or only requires static design files)
* **Relevant Step-Specific Config**: `ERROR_ON_DISCONNECTED_PINS`

##### Flow Python Integration
```python
# Add this step to your flow's sequential Steps list:
class CustomRun(SequentialFlow):
    Steps = [
        ...  # prior steps
        Checker.DisconnectedPins,
        ...  # subsequent steps
    ]
```
##### Configuration Example
```python
# Configure behavior in the flow's config dict:
config = {
    "ERROR_ON_DISCONNECTED_PINS": True,  # (bool) Checks for disconnected instance pins after detailed routing and quits immedi...
}
```

#### `Checker.HoldViolations`
Raises a deferred error if NotImplemented (metric: ``timing__hold_vio__count``) are >= 0. Doesn't raise an error depending on error_on_var if defined.

* **Inputs**: None
* **Outputs**: None
* **Precursor Steps (Dependencies)**:
None (typically runs at the start of the flow, or only requires static design files)
* **Relevant Step-Specific Config**: `HOLD_VIOLATION_CORNERS`, `TIMING_VIOLATION_CORNERS`

##### Flow Python Integration
```python
# Add this step to your flow's sequential Steps list:
class CustomRun(SequentialFlow):
    Steps = [
        ...  # prior steps
        Checker.HoldViolations,
        ...  # subsequent steps
    ]
```
##### Configuration Example
```python
# Configure behavior in the flow's config dict:
config = {
    "TIMING_VIOLATION_CORNERS": None,  # (List[str]) A list of wildcards matching IPVT corners to use during checking for timing v...
    "HOLD_VIOLATION_CORNERS": ['*'],  # (Optional[List[str]]) A list of wildcards matching IPVT corners to use during checking for hold vio...
}
```

#### `Checker.IllegalOverlap`
Raises a deferred error if Magic Illegal Overlap errors (metric: ``magic__illegal_overlap__count``) are >= 0. Doesn't raise an error depending on error_on_var if defined.

* **Inputs**: None
* **Outputs**: None
* **Precursor Steps (Dependencies)**:
None (typically runs at the start of the flow, or only requires static design files)
* **Relevant Step-Specific Config**: `ERROR_ON_ILLEGAL_OVERLAPS`

##### Flow Python Integration
```python
# Add this step to your flow's sequential Steps list:
class CustomRun(SequentialFlow):
    Steps = [
        ...  # prior steps
        Checker.IllegalOverlap,
        ...  # subsequent steps
    ]
```
##### Configuration Example
```python
# Configure behavior in the flow's config dict:
config = {
    "ERROR_ON_ILLEGAL_OVERLAPS": True,  # (bool) Checks for illegal overlaps during Magic extraction. In some cases, these imp...
}
```

#### `Checker.KLayoutAntenna`
Raises a deferred error if KLayout antenna errors (metric: ``klayout__antenna_error__count``) are >= 0. Doesn't raise an error depending on error_on_var if defined.

* **Inputs**: None
* **Outputs**: None
* **Precursor Steps (Dependencies)**:
None (typically runs at the start of the flow, or only requires static design files)
* **Relevant Step-Specific Config**: `ERROR_ON_KLAYOUT_ANTENNA`

##### Flow Python Integration
```python
# Add this step to your flow's sequential Steps list:
class CustomRun(SequentialFlow):
    Steps = [
        ...  # prior steps
        Checker.KLayoutAntenna,
        ...  # subsequent steps
    ]
```
##### Configuration Example
```python
# Configure behavior in the flow's config dict:
config = {
    "ERROR_ON_KLAYOUT_ANTENNA": True,  # (bool) Checks for antenna violations after KLayout antenna check is executed and exi...
}
```

#### `Checker.KLayoutDRC`
Raises a deferred error if KLayout DRC errors (metric: ``klayout__drc_error__count``) are >= 0. Doesn't raise an error depending on error_on_var if defined.

* **Inputs**: None
* **Outputs**: None
* **Precursor Steps (Dependencies)**:
None (typically runs at the start of the flow, or only requires static design files)
* **Relevant Step-Specific Config**: `ERROR_ON_KLAYOUT_DRC`

##### Flow Python Integration
```python
# Add this step to your flow's sequential Steps list:
class CustomRun(SequentialFlow):
    Steps = [
        ...  # prior steps
        Checker.KLayoutDRC,
        ...  # subsequent steps
    ]
```
##### Configuration Example
```python
# Configure behavior in the flow's config dict:
config = {
    "ERROR_ON_KLAYOUT_DRC": True,  # (bool) Checks for DRC violations after KLayout DRC is executed and exits the flow if...
}
```

#### `Checker.KLayoutDensity`
Raises a deferred error if KLayout density errors (metric: ``klayout__density_error__count``) are >= 0. Doesn't raise an error depending on error_on_var if defined.

* **Inputs**: None
* **Outputs**: None
* **Precursor Steps (Dependencies)**:
None (typically runs at the start of the flow, or only requires static design files)
* **Relevant Step-Specific Config**: `ERROR_ON_KLAYOUT_DENSITY`

##### Flow Python Integration
```python
# Add this step to your flow's sequential Steps list:
class CustomRun(SequentialFlow):
    Steps = [
        ...  # prior steps
        Checker.KLayoutDensity,
        ...  # subsequent steps
    ]
```
##### Configuration Example
```python
# Configure behavior in the flow's config dict:
config = {
    "ERROR_ON_KLAYOUT_DENSITY": True,  # (bool) Checks for density violations after KLayout density check is executed and exi...
}
```

#### `Checker.LVS`
Raises a deferred error if LVS errors (metric: ``design__lvs_error__count``) are >= 0. Doesn't raise an error depending on error_on_var if defined.

* **Inputs**: None
* **Outputs**: None
* **Precursor Steps (Dependencies)**:
None (typically runs at the start of the flow, or only requires static design files)
* **Relevant Step-Specific Config**: `ERROR_ON_LVS_ERROR`

##### Flow Python Integration
```python
# Add this step to your flow's sequential Steps list:
class CustomRun(SequentialFlow):
    Steps = [
        ...  # prior steps
        Checker.LVS,
        ...  # subsequent steps
    ]
```
##### Configuration Example
```python
# Configure behavior in the flow's config dict:
config = {
    "ERROR_ON_LVS_ERROR": True,  # (bool) Checks for LVS errors after Netgen is executed. If any exist, it raises an er...
}
```

#### `Checker.LintErrors`
Raises an immediate error if Lint errors (metric: ``design__lint_error__count``) are >= 0. Doesn't raise an error depending on error_on_var if defined.

* **Inputs**: None
* **Outputs**: None
* **Precursor Steps (Dependencies)**:
None (typically runs at the start of the flow, or only requires static design files)
* **Relevant Step-Specific Config**: `ERROR_ON_LINTER_ERRORS`

##### Flow Python Integration
```python
# Add this step to your flow's sequential Steps list:
class CustomRun(SequentialFlow):
    Steps = [
        ...  # prior steps
        Checker.LintErrors,
        ...  # subsequent steps
    ]
```
##### Configuration Example
```python
# Configure behavior in the flow's config dict:
config = {
    "ERROR_ON_LINTER_ERRORS": True,  # (bool) Quit immediately on any linter errors.
}
```

#### `Checker.LintTimingConstructs`
Raises an immediate error if Lint Timing Errors (metric: ``design__lint_timing_construct__count``) are >= 0. Doesn't raise an error depending on error_on_var if defined.

* **Inputs**: None
* **Outputs**: None
* **Precursor Steps (Dependencies)**:
None (typically runs at the start of the flow, or only requires static design files)
* **Relevant Step-Specific Config**: `ERROR_ON_LINTER_TIMING_CONSTRUCTS`

##### Flow Python Integration
```python
# Add this step to your flow's sequential Steps list:
class CustomRun(SequentialFlow):
    Steps = [
        ...  # prior steps
        Checker.LintTimingConstructs,
        ...  # subsequent steps
    ]
```
##### Configuration Example
```python
# Configure behavior in the flow's config dict:
config = {
    "ERROR_ON_LINTER_TIMING_CONSTRUCTS": True,  # (bool) Quit immediately on any discovered timing constructs during linting.
}
```

#### `Checker.LintWarnings`
Raises an immediate error if Lint warnings (metric: ``design__lint_warning__count``) are >= 0. Doesn't raise an error depending on error_on_var if defined.

* **Inputs**: None
* **Outputs**: None
* **Precursor Steps (Dependencies)**:
None (typically runs at the start of the flow, or only requires static design files)
* **Relevant Step-Specific Config**: `ERROR_ON_LINTER_WARNINGS`

##### Flow Python Integration
```python
# Add this step to your flow's sequential Steps list:
class CustomRun(SequentialFlow):
    Steps = [
        ...  # prior steps
        Checker.LintWarnings,
        ...  # subsequent steps
    ]
```
##### Configuration Example
```python
# Configure behavior in the flow's config dict:
config = {
    "ERROR_ON_LINTER_WARNINGS": False,  # (bool) Raise an error immediately on any linter warnings.
}
```

#### `Checker.MagicDRC`
Raises a deferred error if Magic DRC errors (metric: ``magic__drc_error__count``) are >= 0. Doesn't raise an error depending on error_on_var if defined.

* **Inputs**: None
* **Outputs**: None
* **Precursor Steps (Dependencies)**:
None (typically runs at the start of the flow, or only requires static design files)
* **Relevant Step-Specific Config**: `ERROR_ON_MAGIC_DRC`

##### Flow Python Integration
```python
# Add this step to your flow's sequential Steps list:
class CustomRun(SequentialFlow):
    Steps = [
        ...  # prior steps
        Checker.MagicDRC,
        ...  # subsequent steps
    ]
```
##### Configuration Example
```python
# Configure behavior in the flow's config dict:
config = {
    "ERROR_ON_MAGIC_DRC": True,  # (bool) Checks for DRC violations after magic DRC is executed and exits the flow if a...
}
```

#### `Checker.MaxCapViolations`
Raises a deferred error if NotImplemented (metric: ``design__max_cap_violation__count``) are >= 0. Doesn't raise an error depending on error_on_var if defined.

* **Inputs**: None
* **Outputs**: None
* **Precursor Steps (Dependencies)**:
None (typically runs at the start of the flow, or only requires static design files)
* **Relevant Step-Specific Config**: `MAX_CAP_VIOLATION_CORNERS`, `TIMING_VIOLATION_CORNERS`

##### Flow Python Integration
```python
# Add this step to your flow's sequential Steps list:
class CustomRun(SequentialFlow):
    Steps = [
        ...  # prior steps
        Checker.MaxCapViolations,
        ...  # subsequent steps
    ]
```
##### Configuration Example
```python
# Configure behavior in the flow's config dict:
config = {
    "TIMING_VIOLATION_CORNERS": None,  # (List[str]) A list of wildcards matching IPVT corners to use during checking for timing v...
    "MAX_CAP_VIOLATION_CORNERS": [''],  # (Optional[List[str]]) A list of wildcards matching IPVT corners to use during checking for max cap ...
}
```

#### `Checker.MaxSlewViolations`
Raises a deferred error if NotImplemented (metric: ``design__max_slew_violation__count``) are >= 0. Doesn't raise an error depending on error_on_var if defined.

* **Inputs**: None
* **Outputs**: None
* **Precursor Steps (Dependencies)**:
None (typically runs at the start of the flow, or only requires static design files)
* **Relevant Step-Specific Config**: `MAX_SLEW_VIOLATION_CORNERS`, `TIMING_VIOLATION_CORNERS`

##### Flow Python Integration
```python
# Add this step to your flow's sequential Steps list:
class CustomRun(SequentialFlow):
    Steps = [
        ...  # prior steps
        Checker.MaxSlewViolations,
        ...  # subsequent steps
    ]
```
##### Configuration Example
```python
# Configure behavior in the flow's config dict:
config = {
    "TIMING_VIOLATION_CORNERS": None,  # (List[str]) A list of wildcards matching IPVT corners to use during checking for timing v...
    "MAX_SLEW_VIOLATION_CORNERS": [''],  # (Optional[List[str]]) A list of wildcards matching IPVT corners to use during checking for max slew...
}
```

#### `Checker.NetlistAssignStatements`
Raises a StepError if the Netlist has an ``assign`` statement in it.

``assign`` statements are known to cause bugs in some PnR tools.

* **Inputs**: `nl` (Verilog Netlist, `.nl.v`)
* **Outputs**: None
* **Precursor Steps (Dependencies)**:
* Requires `nl` (Verilog Netlist, `.nl.v`), produced by: `Yosys.Synthesis`, `OpenROAD.Floorplan`, `Odb.DiodesOnPorts` (or other steps in Odb, OpenROAD, Yosys)
* **Relevant Step-Specific Config**: `ERROR_ON_NL_ASSIGN_STATEMENTS`

##### Flow Python Integration
```python
# Add this step to your flow's sequential Steps list:
class CustomRun(SequentialFlow):
    Steps = [
        ...  # prior steps
        Checker.NetlistAssignStatements,
        ...  # subsequent steps
    ]
```
##### Configuration Example
```python
# Configure behavior in the flow's config dict:
config = {
    "ERROR_ON_NL_ASSIGN_STATEMENTS": True,  # (bool) Whether to emit an error or simply warn about the existence
}
```

#### `Checker.PowerGridViolations`
Raises a deferred error if power grid violations (as reported by OpenROAD PSM- you may ignore these if LVS passes) (metric: ``design__power_grid_violation__count``) are >= 0. Doesn't raise an error depending on error_on_var if defined.

* **Inputs**: None
* **Outputs**: None
* **Precursor Steps (Dependencies)**:
None (typically runs at the start of the flow, or only requires static design files)
* **Relevant Step-Specific Config**: `ERROR_ON_PDN_VIOLATIONS`

##### Flow Python Integration
```python
# Add this step to your flow's sequential Steps list:
class CustomRun(SequentialFlow):
    Steps = [
        ...  # prior steps
        Checker.PowerGridViolations,
        ...  # subsequent steps
    ]
```
##### Configuration Example
```python
# Configure behavior in the flow's config dict:
config = {
    "ERROR_ON_PDN_VIOLATIONS": True,  # (bool) Checks for unconnected nodes in the power grid. If any exists, an error is ra...
}
```

#### `Checker.SetupViolations`
Raises a deferred error if NotImplemented (metric: ``timing__setup_vio__count``) are >= 0. Doesn't raise an error depending on error_on_var if defined.

* **Inputs**: None
* **Outputs**: None
* **Precursor Steps (Dependencies)**:
None (typically runs at the start of the flow, or only requires static design files)
* **Relevant Step-Specific Config**: `SETUP_VIOLATION_CORNERS`, `TIMING_VIOLATION_CORNERS`

##### Flow Python Integration
```python
# Add this step to your flow's sequential Steps list:
class CustomRun(SequentialFlow):
    Steps = [
        ...  # prior steps
        Checker.SetupViolations,
        ...  # subsequent steps
    ]
```
##### Configuration Example
```python
# Configure behavior in the flow's config dict:
config = {
    "TIMING_VIOLATION_CORNERS": None,  # (List[str]) A list of wildcards matching IPVT corners to use during checking for timing v...
    "SETUP_VIOLATION_CORNERS": None,  # (Optional[List[str]]) A list of wildcards matching IPVT corners to use during checking for setup vi...
}
```

#### `Checker.TrDRC`
Raises a deferred error if Routing DRC errors (metric: ``route__drc_errors``) are >= 0. Doesn't raise an error depending on error_on_var if defined.

* **Inputs**: None
* **Outputs**: None
* **Precursor Steps (Dependencies)**:
None (typically runs at the start of the flow, or only requires static design files)
* **Relevant Step-Specific Config**: `ERROR_ON_TR_DRC`

##### Flow Python Integration
```python
# Add this step to your flow's sequential Steps list:
class CustomRun(SequentialFlow):
    Steps = [
        ...  # prior steps
        Checker.TrDRC,
        ...  # subsequent steps
    ]
```
##### Configuration Example
```python
# Configure behavior in the flow's config dict:
config = {
    "ERROR_ON_TR_DRC": True,  # (bool) Checks for DRC violations after routing and exits the flow if any was found.
}
```

#### `Checker.WireLength`
Raises a deferred error if Threshold-surpassing long wires (metric: ``route__wirelength__max``) are >= the threshold specified in the configuration file.. Doesn't raise an error depending on error_on_var if defined.

* **Inputs**: None
* **Outputs**: None
* **Precursor Steps (Dependencies)**:
None (typically runs at the start of the flow, or only requires static design files)
* **Relevant Step-Specific Config**: `ERROR_ON_LONG_WIRE`, `WIRE_LENGTH_THRESHOLD`

##### Flow Python Integration
```python
# Add this step to your flow's sequential Steps list:
class CustomRun(SequentialFlow):
    Steps = [
        ...  # prior steps
        Checker.WireLength,
        ...  # subsequent steps
    ]
```
##### Configuration Example
```python
# Configure behavior in the flow's config dict:
config = {
    "ERROR_ON_LONG_WIRE": True,  # (bool) Checks if any wire length exceeds the threshold set in the PDK. If so, an err...
    "WIRE_LENGTH_THRESHOLD": None,  # (Optional[decimal.Decimal]) A value above which wire lengths generate warnings.
}
```

#### `Checker.XOR`
Raises a deferred error if XOR differences (metric: ``design__xor_difference__count``) are >= 0. Doesn't raise an error depending on error_on_var if defined.

* **Inputs**: None
* **Outputs**: None
* **Precursor Steps (Dependencies)**:
None (typically runs at the start of the flow, or only requires static design files)
* **Relevant Step-Specific Config**: `ERROR_ON_XOR_ERROR`

##### Flow Python Integration
```python
# Add this step to your flow's sequential Steps list:
class CustomRun(SequentialFlow):
    Steps = [
        ...  # prior steps
        Checker.XOR,
        ...  # subsequent steps
    ]
```
##### Configuration Example
```python
# Configure behavior in the flow's config dict:
config = {
    "ERROR_ON_XOR_ERROR": True,  # (bool) Checks for geometric differences between the Magic and KLayout stream-outs. I...
}
```

#### `Checker.YosysSynthChecks`
Raises an immediate error if Yosys check errors (metric: ``synthesis__check_error__count``) are >= 0. Doesn't raise an error depending on error_on_var if defined.

* **Inputs**: None
* **Outputs**: None
* **Precursor Steps (Dependencies)**:
None (typically runs at the start of the flow, or only requires static design files)
* **Relevant Step-Specific Config**: `ERROR_ON_SYNTH_CHECKS`

##### Flow Python Integration
```python
# Add this step to your flow's sequential Steps list:
class CustomRun(SequentialFlow):
    Steps = [
        ...  # prior steps
        Checker.YosysSynthChecks,
        ...  # subsequent steps
    ]
```
##### Configuration Example
```python
# Configure behavior in the flow's config dict:
config = {
    "ERROR_ON_SYNTH_CHECKS": True,  # (bool) Quits the flow immediately if one or more synthesis check errors are flagged....
}
```

#### `Checker.YosysUnmappedCells`
Raises an immediate error if Unmapped Yosys instances (metric: ``design__instance_unmapped__count``) are >= 0. Doesn't raise an error depending on error_on_var if defined.

* **Inputs**: None
* **Outputs**: None
* **Precursor Steps (Dependencies)**:
None (typically runs at the start of the flow, or only requires static design files)
* **Relevant Step-Specific Config**: `ERROR_ON_UNMAPPED_CELLS`

##### Flow Python Integration
```python
# Add this step to your flow's sequential Steps list:
class CustomRun(SequentialFlow):
    Steps = [
        ...  # prior steps
        Checker.YosysUnmappedCells,
        ...  # subsequent steps
    ]
```
##### Configuration Example
```python
# Configure behavior in the flow's config dict:
config = {
    "ERROR_ON_UNMAPPED_CELLS": True,  # (bool) Checks for unmapped cells after synthesis and quits immediately if so.
}
```

### KLayout Steps

#### `KLayout.Antenna`
Runs the antenna check on the GDS.

* **Inputs**: `gds` (GDSII Stream, `.gds`)
* **Outputs**: None
* **Precursor Steps (Dependencies)**:
* Requires `gds` (GDSII Stream, `.gds`), produced by: `Magic.StreamOut`, `KLayout.StreamOut` (or other steps in KLayout, Magic)
* **Relevant Step-Specific Config**: `KLAYOUT_ANTENNA_OPTIONS`, `KLAYOUT_ANTENNA_RUNSET`, `KLAYOUT_DEF_LAYER_MAP`, `KLAYOUT_PROPERTIES`, `KLAYOUT_TECH`

##### Flow Python Integration
```python
# Add this step to your flow's sequential Steps list:
class CustomRun(SequentialFlow):
    Steps = [
        ...  # prior steps
        KLayout.Antenna,
        ...  # subsequent steps
    ]
```
##### Configuration Example
```python
# Configure behavior in the flow's config dict:
config = {
    "KLAYOUT_TECH": None,  # (librelane.common.types.Path) A path to the KLayout layer technology (.lyt) file.
    "KLAYOUT_PROPERTIES": None,  # (librelane.common.types.Path) A path to the KLayout layer properties (.lyp) file.
    "KLAYOUT_DEF_LAYER_MAP": None,  # (librelane.common.types.Path) A path to the KLayout LEF/DEF layer mapping (.map) file.
    "KLAYOUT_ANTENNA_RUNSET": None,  # (Optional[librelane.common.types.Path]) A path to KLayout antenna runset.
    "KLAYOUT_ANTENNA_OPTIONS": None,  # (Optional[Dict[str, Union[bool, int, str]]]) Options passed directly to the KLayout density runset. They vary from one PDK...
}
```

#### `KLayout.DRC`
Runs DRC using KLayout.

Unlike most steps, the KLayout scripts vary quite wildly by PDK. If a PDK
is not supported by this step, it will simply be skipped.

Currently, only sky130A and sky130B are supported.

* **Inputs**: `gds` (GDSII Stream, `.gds`)
* **Outputs**: None
* **Precursor Steps (Dependencies)**:
* Requires `gds` (GDSII Stream, `.gds`), produced by: `Magic.StreamOut`, `KLayout.StreamOut` (or other steps in KLayout, Magic)
* **Relevant Step-Specific Config**: `KLAYOUT_DEF_LAYER_MAP`, `KLAYOUT_DRC_OPTIONS`, `KLAYOUT_DRC_RUNSET`, `KLAYOUT_DRC_THREADS`, `KLAYOUT_PROPERTIES`, `KLAYOUT_TECH`

##### Flow Python Integration
```python
# Add this step to your flow's sequential Steps list:
class CustomRun(SequentialFlow):
    Steps = [
        ...  # prior steps
        KLayout.DRC,
        ...  # subsequent steps
    ]
```
##### Configuration Example
```python
# Configure behavior in the flow's config dict:
config = {
    "KLAYOUT_TECH": None,  # (librelane.common.types.Path) A path to the KLayout layer technology (.lyt) file.
    "KLAYOUT_PROPERTIES": None,  # (librelane.common.types.Path) A path to the KLayout layer properties (.lyp) file.
    "KLAYOUT_DEF_LAYER_MAP": None,  # (librelane.common.types.Path) A path to the KLayout LEF/DEF layer mapping (.map) file.
    "KLAYOUT_DRC_RUNSET": None,  # (Optional[librelane.common.types.Path]) A path to KLayout DRC runset.
    "KLAYOUT_DRC_OPTIONS": None,  # (Optional[Dict[str, Union[bool, int, str]]]) Options passed directly to the KLayout DRC runset. They vary from one PDK to ...
    "KLAYOUT_DRC_THREADS": None,  # (Optional[int]) Specifies the number of threads to be used in KLayout DRC.If unset, this will...
}
```

#### `KLayout.Density`
Runs the density check on the GDS.

* **Inputs**: `gds` (GDSII Stream, `.gds`)
* **Outputs**: None
* **Precursor Steps (Dependencies)**:
* Requires `gds` (GDSII Stream, `.gds`), produced by: `Magic.StreamOut`, `KLayout.StreamOut` (or other steps in KLayout, Magic)
* **Relevant Step-Specific Config**: `KLAYOUT_DEF_LAYER_MAP`, `KLAYOUT_DENSITY_OPTIONS`, `KLAYOUT_DENSITY_RUNSET`, `KLAYOUT_DENSITY_THREADS`, `KLAYOUT_PROPERTIES`, `KLAYOUT_TECH`

##### Flow Python Integration
```python
# Add this step to your flow's sequential Steps list:
class CustomRun(SequentialFlow):
    Steps = [
        ...  # prior steps
        KLayout.Density,
        ...  # subsequent steps
    ]
```
##### Configuration Example
```python
# Configure behavior in the flow's config dict:
config = {
    "KLAYOUT_TECH": None,  # (librelane.common.types.Path) A path to the KLayout layer technology (.lyt) file.
    "KLAYOUT_PROPERTIES": None,  # (librelane.common.types.Path) A path to the KLayout layer properties (.lyp) file.
    "KLAYOUT_DEF_LAYER_MAP": None,  # (librelane.common.types.Path) A path to the KLayout LEF/DEF layer mapping (.map) file.
    "KLAYOUT_DENSITY_RUNSET": None,  # (Optional[librelane.common.types.Path]) A path to KLayout density runset.
    "KLAYOUT_DENSITY_OPTIONS": None,  # (Optional[Dict[str, Union[bool, int, str]]]) Options passed directly to the KLayout density runset. They vary from one PDK...
    "KLAYOUT_DENSITY_THREADS": None,  # (Optional[int]) Specifies the number of threads to be used in KLayout density check.If unset,...
}
```

#### `KLayout.Filler`
Generates the filler cells according to the design rules and adds them to the GDS.

* **Inputs**: `gds` (GDSII Stream, `.gds`)
* **Outputs**: `gds` (GDSII Stream, `.gds`)
* **Precursor Steps (Dependencies)**:
* Requires `gds` (GDSII Stream, `.gds`), produced by: `Magic.StreamOut`, `KLayout.StreamOut` (or other steps in KLayout, Magic)
* **Relevant Step-Specific Config**: `KLAYOUT_DEF_LAYER_MAP`, `KLAYOUT_FILLER_OPTIONS`, `KLAYOUT_FILLER_SCRIPT`, `KLAYOUT_PROPERTIES`, `KLAYOUT_TECH`

##### Flow Python Integration
```python
# Add this step to your flow's sequential Steps list:
class CustomRun(SequentialFlow):
    Steps = [
        ...  # prior steps
        KLayout.Filler,
        ...  # subsequent steps
    ]
```
##### Configuration Example
```python
# Configure behavior in the flow's config dict:
config = {
    "KLAYOUT_TECH": None,  # (librelane.common.types.Path) A path to the KLayout layer technology (.lyt) file.
    "KLAYOUT_PROPERTIES": None,  # (librelane.common.types.Path) A path to the KLayout layer properties (.lyp) file.
    "KLAYOUT_DEF_LAYER_MAP": None,  # (librelane.common.types.Path) A path to the KLayout LEF/DEF layer mapping (.map) file.
    "KLAYOUT_FILLER_SCRIPT": None,  # (Optional[librelane.common.types.Path]) A path to KLayout filler script.
    "KLAYOUT_FILLER_OPTIONS": None,  # (Optional[Dict[str, Union[bool, int, str]]]) Options passed directly to the KLayout filler script. They vary from one PDK ...
}
```

#### `KLayout.LVS`
No description available.

* **Inputs**: `cdl` (Circuit Design Language, `.cdl`), `gds` (GDSII Stream, `.gds`)
* **Outputs**: `spice` (Simulation Program with Integrated Circuit Emphasis, `.spice`)
* **Precursor Steps (Dependencies)**:
* Requires `cdl` (Circuit Design Language, `.cdl`), produced by: `OpenROAD.WriteCDL`
* Requires `gds` (GDSII Stream, `.gds`), produced by: `Magic.StreamOut`, `KLayout.StreamOut` (or other steps in KLayout, Magic)
* **Relevant Step-Specific Config**: `KLAYOUT_DEF_LAYER_MAP`, `KLAYOUT_LVS_OPTIONS`, `KLAYOUT_LVS_SCRIPT`, `KLAYOUT_PROPERTIES`, `KLAYOUT_TECH`

##### Flow Python Integration
```python
# Add this step to your flow's sequential Steps list:
class CustomRun(SequentialFlow):
    Steps = [
        ...  # prior steps
        KLayout.LVS,
        ...  # subsequent steps
    ]
```
##### Configuration Example
```python
# Configure behavior in the flow's config dict:
config = {
    "KLAYOUT_TECH": None,  # (librelane.common.types.Path) A path to the KLayout layer technology (.lyt) file.
    "KLAYOUT_PROPERTIES": None,  # (librelane.common.types.Path) A path to the KLayout layer properties (.lyp) file.
    "KLAYOUT_DEF_LAYER_MAP": None,  # (librelane.common.types.Path) A path to the KLayout LEF/DEF layer mapping (.map) file.
    "KLAYOUT_LVS_SCRIPT": None,  # (Optional[librelane.common.types.Path]) A path to KLayout LVS script.
    "KLAYOUT_LVS_OPTIONS": None,  # (Optional[Dict[str, Union[bool, int, str]]]) Options passed directly to the KLayout LVS script. They vary from one PDK to ...
}
```

#### `KLayout.OpenGUI`
Opens the DEF view in the KLayout GUI, with layers loaded and mapped
properly. Useful to inspect ``.klayout.xml`` database files and the like.

* **Inputs**: `def` (Design Exchange Format, `.def`)
* **Outputs**: None
* **Precursor Steps (Dependencies)**:
* Requires `def` (Design Exchange Format, `.def`), produced by: `OpenROAD.Floorplan`, `Odb.ApplyDEFTemplate` (or other steps in Odb, OpenROAD)
* **Relevant Step-Specific Config**: `KLAYOUT_DEF_LAYER_MAP`, `KLAYOUT_EDITOR_MODE`, `KLAYOUT_GUI_USE_GDS`, `KLAYOUT_PROPERTIES`, `KLAYOUT_TECH`

##### Flow Python Integration
```python
# Add this step to your flow's sequential Steps list:
class CustomRun(SequentialFlow):
    Steps = [
        ...  # prior steps
        KLayout.OpenGUI,
        ...  # subsequent steps
    ]
```
##### Configuration Example
```python
# Configure behavior in the flow's config dict:
config = {
    "KLAYOUT_TECH": None,  # (librelane.common.types.Path) A path to the KLayout layer technology (.lyt) file.
    "KLAYOUT_PROPERTIES": None,  # (librelane.common.types.Path) A path to the KLayout layer properties (.lyp) file.
    "KLAYOUT_DEF_LAYER_MAP": None,  # (librelane.common.types.Path) A path to the KLayout LEF/DEF layer mapping (.map) file.
    "KLAYOUT_EDITOR_MODE": False,  # (bool) Whether to run the KLayout GUI in editor mode or in viewer mode.
    "KLAYOUT_GUI_USE_GDS": True,  # (bool) Whether to prioritize GDS (if found) when running this step.
}
```

#### `KLayout.Render`
Renders a PNG of the layout using KLayout.

DEF is required as an input, but if a GDS-II view
exists in the input state, it will be used instead.

* **Inputs**: `def` (Design Exchange Format, `.def`)
* **Outputs**: None
* **Precursor Steps (Dependencies)**:
* Requires `def` (Design Exchange Format, `.def`), produced by: `OpenROAD.Floorplan`, `Odb.ApplyDEFTemplate` (or other steps in Odb, OpenROAD)
* **Relevant Step-Specific Config**: `KLAYOUT_DEF_LAYER_MAP`, `KLAYOUT_PROPERTIES`, `KLAYOUT_RENDER_BACKGROUND_COLOR`, `KLAYOUT_RENDER_GRID_VISBLE`, `KLAYOUT_RENDER_OVERSAMPLING`, `KLAYOUT_RENDER_RESOLUTION`, `KLAYOUT_RENDER_SHOW_RULER`, `KLAYOUT_RENDER_TEXT_VISIBLE`, `KLAYOUT_TECH`

##### Flow Python Integration
```python
# Add this step to your flow's sequential Steps list:
class CustomRun(SequentialFlow):
    Steps = [
        ...  # prior steps
        KLayout.Render,
        ...  # subsequent steps
    ]
```
##### Configuration Example
```python
# Configure behavior in the flow's config dict:
config = {
    "KLAYOUT_TECH": None,  # (librelane.common.types.Path) A path to the KLayout layer technology (.lyt) file.
    "KLAYOUT_PROPERTIES": None,  # (librelane.common.types.Path) A path to the KLayout layer properties (.lyp) file.
    "KLAYOUT_DEF_LAYER_MAP": None,  # (librelane.common.types.Path) A path to the KLayout LEF/DEF layer mapping (.map) file.
    "KLAYOUT_RENDER_GRID_VISBLE": False,  # (bool) Render the grid in the image.
    "KLAYOUT_RENDER_SHOW_RULER": False,  # (bool) Enable the ruler in the image.
    "KLAYOUT_RENDER_BACKGROUND_COLOR": 'white',  # (Literal['white', 'black']) The background color of the image.
    "KLAYOUT_RENDER_TEXT_VISIBLE": False,  # (bool) Enable text in the image.
    "KLAYOUT_RENDER_RESOLUTION": 1000,  # (int) The horizontal resolution of the image in pixel.
    "KLAYOUT_RENDER_OVERSAMPLING": 0,  # (int) The oversampling factor (1..3), or 0 for disabling oversampling.
}
```

#### `KLayout.SealRing`
Adds a seal ring in the correct size to the GDS.

* **Inputs**: `gds` (GDSII Stream, `.gds`)
* **Outputs**: `gds` (GDSII Stream, `.gds`)
* **Precursor Steps (Dependencies)**:
* Requires `gds` (GDSII Stream, `.gds`), produced by: `Magic.StreamOut`, `KLayout.StreamOut` (or other steps in KLayout, Magic)
* **Relevant Step-Specific Config**: `KLAYOUT_DEF_LAYER_MAP`, `KLAYOUT_PROPERTIES`, `KLAYOUT_SEALRING_SCRIPT`, `KLAYOUT_TECH`

##### Flow Python Integration
```python
# Add this step to your flow's sequential Steps list:
class CustomRun(SequentialFlow):
    Steps = [
        ...  # prior steps
        KLayout.SealRing,
        ...  # subsequent steps
    ]
```
##### Configuration Example
```python
# Configure behavior in the flow's config dict:
config = {
    "KLAYOUT_TECH": None,  # (librelane.common.types.Path) A path to the KLayout layer technology (.lyt) file.
    "KLAYOUT_PROPERTIES": None,  # (librelane.common.types.Path) A path to the KLayout layer properties (.lyp) file.
    "KLAYOUT_DEF_LAYER_MAP": None,  # (librelane.common.types.Path) A path to the KLayout LEF/DEF layer mapping (.map) file.
    "KLAYOUT_SEALRING_SCRIPT": None,  # (Optional[librelane.common.types.Path]) A path to KLayout seal ring script.
}
```

#### `KLayout.StreamOut`
Converts DEF views into GDSII streams using KLayout.

The PDK must support KLayout for this step to work, otherwise
it will be skipped.

If ``PRIMARY_GDSII_STREAMOUT_TOOL`` is set to ``"klayout"``, both GDS and KLAYOUT_GDS
will be updated, and if set to another tool, only ``KLAYOUT_GDS`` will be
updated.

* **Inputs**: `def` (Design Exchange Format, `.def`)
* **Outputs**: `gds` (GDSII Stream, `.gds`), `klayout_gds` (GDSII Stream (KLayout), `.klayout.gds`)
* **Precursor Steps (Dependencies)**:
* Requires `def` (Design Exchange Format, `.def`), produced by: `OpenROAD.Floorplan`, `Odb.ApplyDEFTemplate` (or other steps in Odb, OpenROAD)
* **Relevant Step-Specific Config**: `KLAYOUT_CONFLICT_RESOLUTION`, `KLAYOUT_DEF_LAYER_MAP`, `KLAYOUT_PROPERTIES`, `KLAYOUT_TECH`

##### Flow Python Integration
```python
# Add this step to your flow's sequential Steps list:
class CustomRun(SequentialFlow):
    Steps = [
        ...  # prior steps
        KLayout.StreamOut,
        ...  # subsequent steps
    ]
```
##### Configuration Example
```python
# Configure behavior in the flow's config dict:
config = {
    "KLAYOUT_TECH": None,  # (librelane.common.types.Path) A path to the KLayout layer technology (.lyt) file.
    "KLAYOUT_PROPERTIES": None,  # (librelane.common.types.Path) A path to the KLayout layer properties (.lyp) file.
    "KLAYOUT_DEF_LAYER_MAP": None,  # (librelane.common.types.Path) A path to the KLayout LEF/DEF layer mapping (.map) file.
    "KLAYOUT_CONFLICT_RESOLUTION": 'RenameCell',  # (Optional[Literal['AddToCell', 'OverwriteCell', 'RenameCell', 'SkipNewCell']]) Specifies the conflict resolution if a cell name conflict arises.
}
```

#### `KLayout.XOR`
Performs an XOR operation on the Magic and KLayout GDS views. The idea is:
if there's any difference between the GDSII streams between the two tools,
one of them have it wrong and that may lead to ambiguity.

* **Inputs**: `mag_gds` (GDSII Stream (Magic), `.magic.gds`), `klayout_gds` (GDSII Stream (KLayout), `.klayout.gds`)
* **Outputs**: None
* **Precursor Steps (Dependencies)**:
* Requires `mag_gds` (GDSII Stream (Magic), `.magic.gds`), produced by: `Magic.StreamOut`
* Requires `klayout_gds` (GDSII Stream (KLayout), `.klayout.gds`), produced by: `KLayout.StreamOut`
* **Relevant Step-Specific Config**: `KLAYOUT_DEF_LAYER_MAP`, `KLAYOUT_PROPERTIES`, `KLAYOUT_TECH`, `KLAYOUT_XOR_IGNORE_LAYERS`, `KLAYOUT_XOR_THREADS`, `KLAYOUT_XOR_TILE_SIZE`

##### Flow Python Integration
```python
# Add this step to your flow's sequential Steps list:
class CustomRun(SequentialFlow):
    Steps = [
        ...  # prior steps
        KLayout.XOR,
        ...  # subsequent steps
    ]
```
##### Configuration Example
```python
# Configure behavior in the flow's config dict:
config = {
    "KLAYOUT_TECH": None,  # (librelane.common.types.Path) A path to the KLayout layer technology (.lyt) file.
    "KLAYOUT_PROPERTIES": None,  # (librelane.common.types.Path) A path to the KLayout layer properties (.lyp) file.
    "KLAYOUT_DEF_LAYER_MAP": None,  # (librelane.common.types.Path) A path to the KLayout LEF/DEF layer mapping (.map) file.
    "KLAYOUT_XOR_THREADS": None,  # (Optional[int]) Specifies number of threads used in the KLayout XOR check. If unset, this wil...
    "KLAYOUT_XOR_IGNORE_LAYERS": None,  # (Optional[List[str]]) KLayout layers to ignore during XOR operations.
    "KLAYOUT_XOR_TILE_SIZE": None,  # (Optional[int]) The tile size to parallelize the XOR process with.
}
```

### Magic Steps

#### `Magic.DRC`
Performs `design rule checking <https://en.wikipedia.org/wiki/Design_rule_checking>`_
on the GDSII stream using Magic.

This also converts the results to a KLayout database, which can be loaded.

The metrics will be updated with ``magic__drc_error__count``. You can use
`the relevant checker <#Checker.MagicDRC>`_ to quit if that number is
nonzero.

* **Inputs**: `def` (Design Exchange Format, `.def`), `gds` (GDSII Stream, `.gds`)
* **Outputs**: None
* **Precursor Steps (Dependencies)**:
* Requires `def` (Design Exchange Format, `.def`), produced by: `OpenROAD.Floorplan`, `Odb.ApplyDEFTemplate` (or other steps in Odb, OpenROAD)
* Requires `gds` (GDSII Stream, `.gds`), produced by: `Magic.StreamOut`, `KLayout.StreamOut` (or other steps in KLayout, Magic)
* **Relevant Step-Specific Config**: `CELL_MAGLEFS`, `CELL_MAGS`, `MAGICRC`, `MAGIC_CAPTURE_ERRORS`, `MAGIC_DEF_LABELS`, `MAGIC_DEF_NO_BLOCKAGES`, `MAGIC_DRC_MAGLEFS`, `MAGIC_DRC_USE_GDS`, `MAGIC_GDS_FLATGLOB`, `MAGIC_GDS_MERGE`, `MAGIC_GDS_POLYGON_SUBCELLS`, `MAGIC_INCLUDE_GDS_POINTERS`, `MAGIC_PDK_SETUP`, `MAGIC_TECH`

##### Flow Python Integration
```python
# Add this step to your flow's sequential Steps list:
class CustomRun(SequentialFlow):
    Steps = [
        ...  # prior steps
        Magic.DRC,
        ...  # subsequent steps
    ]
```
##### Configuration Example
```python
# Configure behavior in the flow's config dict:
config = {
    "MAGIC_DEF_LABELS": False,  # (bool) A flag to choose whether labels are read with DEF files or not. From magic do...
    "MAGIC_GDS_POLYGON_SUBCELLS": False,  # (bool) A flag to enable polygon subcells in magic for gds read potentially speeding ...
    "MAGIC_GDS_MERGE": True,  # (bool) A flag to enable merging of connected tiles into polygons during gds write. F...
    "MAGIC_DEF_NO_BLOCKAGES": True,  # (bool) If set to true, blockages in DEF files are ignored. Otherwise, they are read ...
    "MAGIC_INCLUDE_GDS_POINTERS": False,  # (bool) A flag to choose whether to include GDS pointers in the generated mag files o...
    "MAGICRC": None,  # (librelane.common.types.Path) A path to the `.magicrc` file which is sourced before running magic in the flow.
    "MAGIC_TECH": None,  # (librelane.common.types.Path) A path to a Magic tech file which, mainly, has DRC rules.
    "MAGIC_PDK_SETUP": None,  # (librelane.common.types.Path) A path to a PDK-specific setup file sourced by `.magicrc`.
    "CELL_MAGS": None,  # (Optional[List[librelane.common.types.Path]]) A list of pre-processed concrete views for cells. Read as a fallback for unde...
    "CELL_MAGLEFS": None,  # (Optional[List[librelane.common.types.Path]]) A list of pre-processed abstract LEF views for cells. Read as a fallback for ...
    "MAGIC_CAPTURE_ERRORS": True,  # (bool) Capture errors print by Magic and quit when a fatal error is encountered. Fat...
    "MAGIC_DRC_USE_GDS": True,  # (bool) A flag to choose whether to run the Magic DRC checks on GDS or not. If not, t...
    "MAGIC_GDS_FLATGLOB": None,  # (Optional[List[str]]) Flatten cells by name pattern on input. May be used to avoid false positive D...
    "MAGIC_DRC_MAGLEFS": None,  # (Optional[List[librelane.common.types.Path]]) A list of pre-processed abstract LEF views for cells. They are read in before...
}
```

#### `Magic.Filler`
Generates the filler cells according to the design rules and adds them to the GDS.

* **Inputs**: `gds` (GDSII Stream, `.gds`)
* **Outputs**: `gds` (GDSII Stream, `.gds`)
* **Precursor Steps (Dependencies)**:
* Requires `gds` (GDSII Stream, `.gds`), produced by: `Magic.StreamOut`, `KLayout.StreamOut` (or other steps in KLayout, Magic)
* **Relevant Step-Specific Config**: `MAGIC_FILLER_OPTIONS`, `MAGIC_FILLER_SCRIPT`

##### Flow Python Integration
```python
# Add this step to your flow's sequential Steps list:
class CustomRun(SequentialFlow):
    Steps = [
        ...  # prior steps
        Magic.Filler,
        ...  # subsequent steps
    ]
```
##### Configuration Example
```python
# Configure behavior in the flow's config dict:
config = {
    "MAGIC_FILLER_SCRIPT": None,  # (Optional[librelane.common.types.Path]) Path to the magic filler script.
    "MAGIC_FILLER_OPTIONS": None,  # (Optional[List[str]]) Options passed directly to the magic filler script.
}
```

#### `Magic.OpenGUI`
Opens the DEF view in the Magic GUI.

* **Inputs**: `def` (Design Exchange Format, `.def`)
* **Outputs**: None
* **Precursor Steps (Dependencies)**:
* Requires `def` (Design Exchange Format, `.def`), produced by: `OpenROAD.Floorplan`, `Odb.ApplyDEFTemplate` (or other steps in Odb, OpenROAD)
* **Relevant Step-Specific Config**: `CELL_MAGLEFS`, `CELL_MAGS`, `MAGICRC`, `MAGIC_CAPTURE_ERRORS`, `MAGIC_DEF_LABELS`, `MAGIC_DEF_NO_BLOCKAGES`, `MAGIC_GDS_MERGE`, `MAGIC_GDS_POLYGON_SUBCELLS`, `MAGIC_GUI_USE_GDS`, `MAGIC_INCLUDE_GDS_POINTERS`, `MAGIC_PDK_SETUP`, `MAGIC_TECH`

##### Flow Python Integration
```python
# Add this step to your flow's sequential Steps list:
class CustomRun(SequentialFlow):
    Steps = [
        ...  # prior steps
        Magic.OpenGUI,
        ...  # subsequent steps
    ]
```
##### Configuration Example
```python
# Configure behavior in the flow's config dict:
config = {
    "MAGIC_DEF_LABELS": False,  # (bool) A flag to choose whether labels are read with DEF files or not. From magic do...
    "MAGIC_GDS_POLYGON_SUBCELLS": False,  # (bool) A flag to enable polygon subcells in magic for gds read potentially speeding ...
    "MAGIC_GDS_MERGE": True,  # (bool) A flag to enable merging of connected tiles into polygons during gds write. F...
    "MAGIC_DEF_NO_BLOCKAGES": True,  # (bool) If set to true, blockages in DEF files are ignored. Otherwise, they are read ...
    "MAGIC_INCLUDE_GDS_POINTERS": False,  # (bool) A flag to choose whether to include GDS pointers in the generated mag files o...
    "MAGICRC": None,  # (librelane.common.types.Path) A path to the `.magicrc` file which is sourced before running magic in the flow.
    "MAGIC_TECH": None,  # (librelane.common.types.Path) A path to a Magic tech file which, mainly, has DRC rules.
    "MAGIC_PDK_SETUP": None,  # (librelane.common.types.Path) A path to a PDK-specific setup file sourced by `.magicrc`.
    "CELL_MAGS": None,  # (Optional[List[librelane.common.types.Path]]) A list of pre-processed concrete views for cells. Read as a fallback for unde...
    "CELL_MAGLEFS": None,  # (Optional[List[librelane.common.types.Path]]) A list of pre-processed abstract LEF views for cells. Read as a fallback for ...
    "MAGIC_CAPTURE_ERRORS": True,  # (bool) Capture errors print by Magic and quit when a fatal error is encountered. Fat...
    "MAGIC_GUI_USE_GDS": True,  # (bool) Whether to prioritize GDS (if found) when running this step.
}
```

#### `Magic.RCX`
Performs a full parasitics extraction (RCX) using Magic. This step is suitable for performing full
analogue post-layout simulation of your design. The SPICE netlist will be flattened, and refer to the raw
n-type/p-type transistor models for your PDK.

If you want to do device level simulation *without* parasitics, this can be done in the
Magic.SpiceExtraction step by setting `MAGIC_EXT_USE_GDS` to True and `MAGIC_EXT_ABSTRACT` to False.

* **Inputs**: `gds` (GDSII Stream, `.gds`)
* **Outputs**: `spice_rcx` (Simulation Program with Integrated Circuit Emphasis (with Resistance and Capacitance eXtraction), `.rcx.spice`)
* **Precursor Steps (Dependencies)**:
* Requires `gds` (GDSII Stream, `.gds`), produced by: `Magic.StreamOut`, `KLayout.StreamOut` (or other steps in KLayout, Magic)
* **Relevant Step-Specific Config**: `CELL_MAGLEFS`, `CELL_MAGS`, `MAGICRC`, `MAGIC_CAPTURE_ERRORS`, `MAGIC_DEF_LABELS`, `MAGIC_DEF_NO_BLOCKAGES`, `MAGIC_GDS_MERGE`, `MAGIC_GDS_POLYGON_SUBCELLS`, `MAGIC_INCLUDE_GDS_POINTERS`, `MAGIC_PDK_SETUP`, `MAGIC_RCX_CTHRESH`, `MAGIC_RCX_DO_CAPACITANCE`, `MAGIC_RCX_DO_RESISTANCE`, `MAGIC_RCX_EXTRACT_STYLE`, `MAGIC_TECH`

##### Flow Python Integration
```python
# Add this step to your flow's sequential Steps list:
class CustomRun(SequentialFlow):
    Steps = [
        ...  # prior steps
        Magic.RCX,
        ...  # subsequent steps
    ]
```
##### Configuration Example
```python
# Configure behavior in the flow's config dict:
config = {
    "MAGIC_DEF_LABELS": False,  # (bool) A flag to choose whether labels are read with DEF files or not. From magic do...
    "MAGIC_GDS_POLYGON_SUBCELLS": False,  # (bool) A flag to enable polygon subcells in magic for gds read potentially speeding ...
    "MAGIC_GDS_MERGE": True,  # (bool) A flag to enable merging of connected tiles into polygons during gds write. F...
    "MAGIC_DEF_NO_BLOCKAGES": True,  # (bool) If set to true, blockages in DEF files are ignored. Otherwise, they are read ...
    "MAGIC_INCLUDE_GDS_POINTERS": False,  # (bool) A flag to choose whether to include GDS pointers in the generated mag files o...
    "MAGICRC": None,  # (librelane.common.types.Path) A path to the `.magicrc` file which is sourced before running magic in the flow.
    "MAGIC_TECH": None,  # (librelane.common.types.Path) A path to a Magic tech file which, mainly, has DRC rules.
    "MAGIC_PDK_SETUP": None,  # (librelane.common.types.Path) A path to a PDK-specific setup file sourced by `.magicrc`.
    "CELL_MAGS": None,  # (Optional[List[librelane.common.types.Path]]) A list of pre-processed concrete views for cells. Read as a fallback for unde...
    "CELL_MAGLEFS": None,  # (Optional[List[librelane.common.types.Path]]) A list of pre-processed abstract LEF views for cells. Read as a fallback for ...
    "MAGIC_CAPTURE_ERRORS": True,  # (bool) Capture errors print by Magic and quit when a fatal error is encountered. Fat...
    "MAGIC_RCX_CTHRESH": 0.1,  # (float) Capacitance threshold value.
    "MAGIC_RCX_DO_CAPACITANCE": True,  # (bool) Whether to extract local capacitance values.
    "MAGIC_RCX_DO_RESISTANCE": True,  # (bool) Whether to perform detailed (i.e. non-lumped) resistance extraction.
    "MAGIC_RCX_EXTRACT_STYLE": 'ngspice()',  # (str) Capacitance extraction corner. For open PDKs, the options are generally: 'ngs...
}
```

#### `Magic.SpiceExtraction`
Extracts a SPICE netlist from the GDSII stream. Used in Layout vs. Schematic
checks.

Note that the resultant SPICE netlist is blackboxed, and *only* suitable for abstract LVS.
If you want to perform a full parasitics extraction (RCX), you should use the Magic.RCX step.

Also, the metrics will be updated with ``magic__illegal_overlap__count``. You can use
`the relevant checker <#Checker.IllegalOverlap>`_ to quit if that number is
nonzero.

* **Inputs**: `gds` (GDSII Stream, `.gds`), `def` (Design Exchange Format, `.def`)
* **Outputs**: `spice` (Simulation Program with Integrated Circuit Emphasis, `.spice`)
* **Precursor Steps (Dependencies)**:
* Requires `gds` (GDSII Stream, `.gds`), produced by: `Magic.StreamOut`, `KLayout.StreamOut` (or other steps in KLayout, Magic)
* Requires `def` (Design Exchange Format, `.def`), produced by: `OpenROAD.Floorplan`, `Odb.ApplyDEFTemplate` (or other steps in Odb, OpenROAD)
* **Relevant Step-Specific Config**: `CELL_MAGLEFS`, `CELL_MAGS`, `MAGICRC`, `MAGIC_CAPTURE_ERRORS`, `MAGIC_DEF_LABELS`, `MAGIC_DEF_NO_BLOCKAGES`, `MAGIC_EXT_ABSTRACT`, `MAGIC_EXT_ABSTRACT_CELLS`, `MAGIC_EXT_SHORT_RESISTOR`, `MAGIC_EXT_UNIQUE`, `MAGIC_EXT_USE_GDS`, `MAGIC_FEEDBACK_CONVERSION_THRESHOLD`, `MAGIC_GDS_MERGE`, `MAGIC_GDS_POLYGON_SUBCELLS`, `MAGIC_INCLUDE_GDS_POINTERS`, `MAGIC_PDK_SETUP`, `MAGIC_TECH`

##### Flow Python Integration
```python
# Add this step to your flow's sequential Steps list:
class CustomRun(SequentialFlow):
    Steps = [
        ...  # prior steps
        Magic.SpiceExtraction,
        ...  # subsequent steps
    ]
```
##### Configuration Example
```python
# Configure behavior in the flow's config dict:
config = {
    "MAGIC_DEF_LABELS": False,  # (bool) A flag to choose whether labels are read with DEF files or not. From magic do...
    "MAGIC_GDS_POLYGON_SUBCELLS": False,  # (bool) A flag to enable polygon subcells in magic for gds read potentially speeding ...
    "MAGIC_GDS_MERGE": True,  # (bool) A flag to enable merging of connected tiles into polygons during gds write. F...
    "MAGIC_DEF_NO_BLOCKAGES": True,  # (bool) If set to true, blockages in DEF files are ignored. Otherwise, they are read ...
    "MAGIC_INCLUDE_GDS_POINTERS": False,  # (bool) A flag to choose whether to include GDS pointers in the generated mag files o...
    "MAGICRC": None,  # (librelane.common.types.Path) A path to the `.magicrc` file which is sourced before running magic in the flow.
    "MAGIC_TECH": None,  # (librelane.common.types.Path) A path to a Magic tech file which, mainly, has DRC rules.
    "MAGIC_PDK_SETUP": None,  # (librelane.common.types.Path) A path to a PDK-specific setup file sourced by `.magicrc`.
    "CELL_MAGS": None,  # (Optional[List[librelane.common.types.Path]]) A list of pre-processed concrete views for cells. Read as a fallback for unde...
    "CELL_MAGLEFS": None,  # (Optional[List[librelane.common.types.Path]]) A list of pre-processed abstract LEF views for cells. Read as a fallback for ...
    "MAGIC_CAPTURE_ERRORS": True,  # (bool) Capture errors print by Magic and quit when a fatal error is encountered. Fat...
    "MAGIC_EXT_USE_GDS": False,  # (bool) A flag to choose whether to use GDS for spice extraction or not. If not, then...
    "MAGIC_EXT_ABSTRACT_CELLS": None,  # (Optional[List[str]]) A list of regular expressions which are matched against the cells of a the de...
    "MAGIC_EXT_UNIQUE": 'all',  # (Literal['all', 'notopports', 'noports', 'none']) Runs `extract unique` with the specified option. The default is "all", and "n...
    "MAGIC_EXT_SHORT_RESISTOR": False,  # (bool) Enables adding resistors to shorts- resolves LVS issues if more than one top-...
    "MAGIC_EXT_ABSTRACT": False,  # (bool) Extracts a SPICE netlist based on black-boxed standard cells and macros (basi...
    "MAGIC_FEEDBACK_CONVERSION_THRESHOLD": 10000,  # (int) If Magic provides more feedback items than this threshold, conversion to KLay...
}
```

#### `Magic.StreamOut`
Converts DEF views into GDSII streams using Magic.

If ``PRIMARY_GDSII_STREAMOUT_TOOL`` is set to ``"magic"``, both GDS and MAG_GDS
will be updated, and if set to another tool, only ``MAG_GDS`` will be
updated.

* **Inputs**: `def` (Design Exchange Format, `.def`)
* **Outputs**: `gds` (GDSII Stream, `.gds`), `mag_gds` (GDSII Stream (Magic), `.magic.gds`), `mag` (Magic VLSI View, `.mag`)
* **Precursor Steps (Dependencies)**:
* Requires `def` (Design Exchange Format, `.def`), produced by: `OpenROAD.Floorplan`, `Odb.ApplyDEFTemplate` (or other steps in Odb, OpenROAD)
* **Relevant Step-Specific Config**: `CELL_MAGLEFS`, `CELL_MAGS`, `MAGICRC`, `MAGIC_CAPTURE_ERRORS`, `MAGIC_DEF_LABELS`, `MAGIC_DEF_NO_BLOCKAGES`, `MAGIC_DISABLE_CIF_INFO`, `MAGIC_GDS_MERGE`, `MAGIC_GDS_POLYGON_SUBCELLS`, `MAGIC_INCLUDE_GDS_POINTERS`, `MAGIC_MACRO_STD_CELL_SOURCE`, `MAGIC_PDK_SETUP`, `MAGIC_TECH`, `MAGIC_ZEROIZE_ORIGIN`

##### Flow Python Integration
```python
# Add this step to your flow's sequential Steps list:
class CustomRun(SequentialFlow):
    Steps = [
        ...  # prior steps
        Magic.StreamOut,
        ...  # subsequent steps
    ]
```
##### Configuration Example
```python
# Configure behavior in the flow's config dict:
config = {
    "MAGIC_DEF_LABELS": False,  # (bool) A flag to choose whether labels are read with DEF files or not. From magic do...
    "MAGIC_GDS_POLYGON_SUBCELLS": False,  # (bool) A flag to enable polygon subcells in magic for gds read potentially speeding ...
    "MAGIC_GDS_MERGE": True,  # (bool) A flag to enable merging of connected tiles into polygons during gds write. F...
    "MAGIC_DEF_NO_BLOCKAGES": True,  # (bool) If set to true, blockages in DEF files are ignored. Otherwise, they are read ...
    "MAGIC_INCLUDE_GDS_POINTERS": False,  # (bool) A flag to choose whether to include GDS pointers in the generated mag files o...
    "MAGICRC": None,  # (librelane.common.types.Path) A path to the `.magicrc` file which is sourced before running magic in the flow.
    "MAGIC_TECH": None,  # (librelane.common.types.Path) A path to a Magic tech file which, mainly, has DRC rules.
    "MAGIC_PDK_SETUP": None,  # (librelane.common.types.Path) A path to a PDK-specific setup file sourced by `.magicrc`.
    "CELL_MAGS": None,  # (Optional[List[librelane.common.types.Path]]) A list of pre-processed concrete views for cells. Read as a fallback for unde...
    "CELL_MAGLEFS": None,  # (Optional[List[librelane.common.types.Path]]) A list of pre-processed abstract LEF views for cells. Read as a fallback for ...
    "MAGIC_CAPTURE_ERRORS": True,  # (bool) Capture errors print by Magic and quit when a fatal error is encountered. Fat...
    "MAGIC_ZEROIZE_ORIGIN": False,  # (bool) A flag to move the layout such that it's origin in the lef generated by magic...
    "MAGIC_DISABLE_CIF_INFO": True,  # (bool) A flag to disable writing Caltech Intermediate Format (CIF) hierarchy and sub...
    "MAGIC_MACRO_STD_CELL_SOURCE": 'macro',  # (Literal['PDK', 'macro']) If set to PDK, magic will use the PDK definition of the STD cells for macros ...
}
```

#### `Magic.WriteLEF`
Writes a LEF view of the design using the GDS using Magic.

* **Inputs**: `gds` (GDSII Stream, `.gds`), `def` (Design Exchange Format, `.def`)
* **Outputs**: `lef` (Library Exchange Format, `.lef`)
* **Precursor Steps (Dependencies)**:
* Requires `gds` (GDSII Stream, `.gds`), produced by: `Magic.StreamOut`, `KLayout.StreamOut` (or other steps in KLayout, Magic)
* Requires `def` (Design Exchange Format, `.def`), produced by: `OpenROAD.Floorplan`, `Odb.ApplyDEFTemplate` (or other steps in Odb, OpenROAD)
* **Relevant Step-Specific Config**: `CELL_MAGLEFS`, `CELL_MAGS`, `MAGICRC`, `MAGIC_CAPTURE_ERRORS`, `MAGIC_DEF_LABELS`, `MAGIC_DEF_NO_BLOCKAGES`, `MAGIC_GDS_MERGE`, `MAGIC_GDS_POLYGON_SUBCELLS`, `MAGIC_INCLUDE_GDS_POINTERS`, `MAGIC_LEF_WRITE_USE_GDS`, `MAGIC_PDK_SETUP`, `MAGIC_TECH`, `MAGIC_WRITE_FULL_LEF`, `MAGIC_WRITE_LEF_PINONLY`

##### Flow Python Integration
```python
# Add this step to your flow's sequential Steps list:
class CustomRun(SequentialFlow):
    Steps = [
        ...  # prior steps
        Magic.WriteLEF,
        ...  # subsequent steps
    ]
```
##### Configuration Example
```python
# Configure behavior in the flow's config dict:
config = {
    "MAGIC_DEF_LABELS": False,  # (bool) A flag to choose whether labels are read with DEF files or not. From magic do...
    "MAGIC_GDS_POLYGON_SUBCELLS": False,  # (bool) A flag to enable polygon subcells in magic for gds read potentially speeding ...
    "MAGIC_GDS_MERGE": True,  # (bool) A flag to enable merging of connected tiles into polygons during gds write. F...
    "MAGIC_DEF_NO_BLOCKAGES": True,  # (bool) If set to true, blockages in DEF files are ignored. Otherwise, they are read ...
    "MAGIC_INCLUDE_GDS_POINTERS": False,  # (bool) A flag to choose whether to include GDS pointers in the generated mag files o...
    "MAGICRC": None,  # (librelane.common.types.Path) A path to the `.magicrc` file which is sourced before running magic in the flow.
    "MAGIC_TECH": None,  # (librelane.common.types.Path) A path to a Magic tech file which, mainly, has DRC rules.
    "MAGIC_PDK_SETUP": None,  # (librelane.common.types.Path) A path to a PDK-specific setup file sourced by `.magicrc`.
    "CELL_MAGS": None,  # (Optional[List[librelane.common.types.Path]]) A list of pre-processed concrete views for cells. Read as a fallback for unde...
    "CELL_MAGLEFS": None,  # (Optional[List[librelane.common.types.Path]]) A list of pre-processed abstract LEF views for cells. Read as a fallback for ...
    "MAGIC_CAPTURE_ERRORS": True,  # (bool) Capture errors print by Magic and quit when a fatal error is encountered. Fat...
    "MAGIC_LEF_WRITE_USE_GDS": False,  # (bool) A flag to choose whether to use GDS for LEF writing. If not, then the extract...
    "MAGIC_WRITE_FULL_LEF": False,  # (bool) A flag to specify whether or not the output LEF should include all shapes ins...
    "MAGIC_WRITE_LEF_PINONLY": False,  # (bool) If true, the LEF write will mark only areas that are port labels as pins, whi...
}
```

### Misc Steps

#### `Misc.LoadBaseSDC`
Loads an SDC file specified as a configuration variable into the state
object unaltered.

This Step exists for legacy compatibility and should not be used
in new flows.

* **Inputs**: None
* **Outputs**: `sdc` (Design Constraints, `.sdc`)
* **Precursor Steps (Dependencies)**:
None (typically runs at the start of the flow, or only requires static design files)

##### Flow Python Integration
```python
# Add this step to your flow's sequential Steps list:
class CustomRun(SequentialFlow):
    Steps = [
        ...  # prior steps
        Misc.LoadBaseSDC,
        ...  # subsequent steps
    ]
```

#### `Misc.ReportManufacturability`
Logs a simple "manufacturability report", i.e., the status of DRC, LVS, and
antenna violations.

* **Inputs**: None
* **Outputs**: None
* **Precursor Steps (Dependencies)**:
None (typically runs at the start of the flow, or only requires static design files)

##### Flow Python Integration
```python
# Add this step to your flow's sequential Steps list:
class CustomRun(SequentialFlow):
    Steps = [
        ...  # prior steps
        Misc.ReportManufacturability,
        ...  # subsequent steps
    ]
```

### Netgen Steps

#### `Netgen.LVS`
Performs `Layout vs. Schematic <https://en.wikipedia.org/wiki/Layout_Versus_Schematic>`_ checks on the extracted SPICE netlist versus.
a Verilog netlist with power connections.

This verifies the following:
* There are no unexpected shorts in the final layout.
* There are no unexpected opens in the final layout.
* All signals are connected correctly.

* **Inputs**: `spice` (Simulation Program with Integrated Circuit Emphasis, `.spice`), `pnl` (Powered Verilog Netlist, `.pnl.v`)
* **Outputs**: None
* **Precursor Steps (Dependencies)**:
* Requires `spice` (Simulation Program with Integrated Circuit Emphasis, `.spice`), produced by: `KLayout.LVS`, `Magic.SpiceExtraction`
* Requires `pnl` (Powered Verilog Netlist, `.pnl.v`), produced by: `OpenROAD.Floorplan`, `Odb.DiodesOnPorts` (or other steps in Odb, OpenROAD)
* **Relevant Step-Specific Config**: `LVS_FLATTEN_CELLS`, `LVS_IGNORE_CELLS`, `LVS_INCLUDE_MARCO_NETLISTS`, `MAGIC_EXT_USE_GDS`, `NETGEN_SETUP`

##### Flow Python Integration
```python
# Add this step to your flow's sequential Steps list:
class CustomRun(SequentialFlow):
    Steps = [
        ...  # prior steps
        Netgen.LVS,
        ...  # subsequent steps
    ]
```
##### Configuration Example
```python
# Configure behavior in the flow's config dict:
config = {
    "MAGIC_EXT_USE_GDS": False,  # (bool) A flag to choose whether to use GDS for spice extraction or not. If not, then...
    "NETGEN_SETUP": None,  # (librelane.common.types.Path) A path to the setup file for Netgen used to configure LVS. If set to None, th...
    "LVS_INCLUDE_MARCO_NETLISTS": False,  # (bool) A flag that enables including the gate-level netlist of macros while running ...
    "LVS_FLATTEN_CELLS": None,  # (Optional[List[str]]) A list of cell names to be flattened while running LVS
    "LVS_IGNORE_CELLS": None,  # (Optional[List[str]]) A list of cell names to be ignored while running LVS
}
```

### Odb Steps

#### `Odb.AddPDNObstructions`
Adds obstructions on metal layers which prevent shapes from being created in
the designated areas.

A soft-duplicate of <#Odb.AddRoutingObstructions>`_ , though this one uses
a different variable name so the obstructions can be restricted for PDN
steps only.

* **Inputs**: `odb` (OpenDB Database, `.odb`)
* **Outputs**: `odb` (OpenDB Database, `.odb`), `def` (Design Exchange Format, `.def`)
* **Precursor Steps (Dependencies)**:
* Requires `odb` (OpenDB Database, `.odb`), produced by: `OpenROAD.Floorplan`, `Odb.ApplyDEFTemplate` (or other steps in Odb, OpenROAD)
* **Relevant Step-Specific Config**: `PDN_OBSTRUCTIONS`

##### Flow Python Integration
```python
# Add this step to your flow's sequential Steps list:
class CustomRun(SequentialFlow):
    Steps = [
        ...  # prior steps
        Odb.AddPDNObstructions,
        ...  # subsequent steps
    ]
```
##### Configuration Example
```python
# Configure behavior in the flow's config dict:
config = {
    "PDN_OBSTRUCTIONS": None,  # (Optional[List[Tuple[str, decimal.Decimal, decimal.Decimal, decimal.Decimal, decimal.Decimal]]]) Add routing obstructions to the design before PDN stage. If set to `None`, th...
}
```

#### `Odb.AddRoutingObstructions`
Adds obstructions on metal layers which prevent shapes from being created in
the designated areas.

* **Inputs**: `odb` (OpenDB Database, `.odb`)
* **Outputs**: `odb` (OpenDB Database, `.odb`), `def` (Design Exchange Format, `.def`)
* **Precursor Steps (Dependencies)**:
* Requires `odb` (OpenDB Database, `.odb`), produced by: `OpenROAD.Floorplan`, `Odb.ApplyDEFTemplate` (or other steps in Odb, OpenROAD)
* **Relevant Step-Specific Config**: `ROUTING_OBSTRUCTIONS`

##### Flow Python Integration
```python
# Add this step to your flow's sequential Steps list:
class CustomRun(SequentialFlow):
    Steps = [
        ...  # prior steps
        Odb.AddRoutingObstructions,
        ...  # subsequent steps
    ]
```
##### Configuration Example
```python
# Configure behavior in the flow's config dict:
config = {
    "ROUTING_OBSTRUCTIONS": None,  # (Optional[List[Tuple[str, decimal.Decimal, decimal.Decimal, decimal.Decimal, decimal.Decimal]]]) Add routing obstructions to the design. If set to `None`, this step is skippe...
}
```

#### `Odb.ApplyDEFTemplate`
Copies the floorplan of a "template" DEF file for a new design, i.e.,
it will copy the die area, core area, and non-power pin names and locations.

* **Inputs**: `odb` (OpenDB Database, `.odb`)
* **Outputs**: `odb` (OpenDB Database, `.odb`), `def` (Design Exchange Format, `.def`)
* **Precursor Steps (Dependencies)**:
* Requires `odb` (OpenDB Database, `.odb`), produced by: `OpenROAD.Floorplan`, `Odb.ApplyDEFTemplate` (or other steps in Odb, OpenROAD)
* **Relevant Step-Specific Config**: `FP_DEF_TEMPLATE`, `FP_TEMPLATE_COPY_POWER_PINS`, `FP_TEMPLATE_MATCH_MODE`

##### Flow Python Integration
```python
# Add this step to your flow's sequential Steps list:
class CustomRun(SequentialFlow):
    Steps = [
        ...  # prior steps
        Odb.ApplyDEFTemplate,
        ...  # subsequent steps
    ]
```
##### Configuration Example
```python
# Configure behavior in the flow's config dict:
config = {
    "FP_DEF_TEMPLATE": None,  # (Optional[librelane.common.types.Path]) Points to the DEF file to be used as a template.
    "FP_TEMPLATE_MATCH_MODE": 'strict',  # (Literal['strict', 'permissive']) Whether to require that the pin set of the DEF template and the design should...
    "FP_TEMPLATE_COPY_POWER_PINS": False,  # (bool) Whether to *always* copy all power pins from the DEF template to the design.
}
```

#### `Odb.CellFrequencyTables`
Creates a number of tables to show the cell frequencies by:

- Cells
- Buffer cells only
- Cell Function*
- Standard Cell Library*

* These tables only return meaningful info with PDKs distributed in the
Open_PDKs format, i.e., all cells are named ``{scl}__{cell_fn}_{size}``.

* **Inputs**: `odb` (OpenDB Database, `.odb`)
* **Outputs**: `odb` (OpenDB Database, `.odb`), `def` (Design Exchange Format, `.def`)
* **Precursor Steps (Dependencies)**:
* Requires `odb` (OpenDB Database, `.odb`), produced by: `OpenROAD.Floorplan`, `Odb.ApplyDEFTemplate` (or other steps in Odb, OpenROAD)

##### Flow Python Integration
```python
# Add this step to your flow's sequential Steps list:
class CustomRun(SequentialFlow):
    Steps = [
        ...  # prior steps
        Odb.CellFrequencyTables,
        ...  # subsequent steps
    ]
```

#### `Odb.CheckDesignAntennaProperties`
Prints warnings if the LEF view of the design is missing antenna information.

* **Inputs**: `odb` (OpenDB Database, `.odb`), `lef` (Library Exchange Format, `.lef`)
* **Outputs**: None
* **Precursor Steps (Dependencies)**:
* Requires `odb` (OpenDB Database, `.odb`), produced by: `OpenROAD.Floorplan`, `Odb.ApplyDEFTemplate` (or other steps in Odb, OpenROAD)
* Requires `lef` (Library Exchange Format, `.lef`), produced by: `Magic.WriteLEF`

##### Flow Python Integration
```python
# Add this step to your flow's sequential Steps list:
class CustomRun(SequentialFlow):
    Steps = [
        ...  # prior steps
        Odb.CheckDesignAntennaProperties,
        ...  # subsequent steps
    ]
```

#### `Odb.CheckMacroAntennaProperties`
Prints warnings if the LEF views of macros are missing antenna information.

* **Inputs**: `odb` (OpenDB Database, `.odb`)
* **Outputs**: None
* **Precursor Steps (Dependencies)**:
* Requires `odb` (OpenDB Database, `.odb`), produced by: `OpenROAD.Floorplan`, `Odb.ApplyDEFTemplate` (or other steps in Odb, OpenROAD)

##### Flow Python Integration
```python
# Add this step to your flow's sequential Steps list:
class CustomRun(SequentialFlow):
    Steps = [
        ...  # prior steps
        Odb.CheckMacroAntennaProperties,
        ...  # subsequent steps
    ]
```

#### `Odb.CustomIOPlacement`
Places I/O pins using a custom script, which uses a "pin order configuration"
file.

Check the reference documentation for the structure of said file.

* **Inputs**: `odb` (OpenDB Database, `.odb`)
* **Outputs**: `odb` (OpenDB Database, `.odb`), `def` (Design Exchange Format, `.def`)
* **Precursor Steps (Dependencies)**:
* Requires `odb` (OpenDB Database, `.odb`), produced by: `OpenROAD.Floorplan`, `Odb.ApplyDEFTemplate` (or other steps in Odb, OpenROAD)
* **Relevant Step-Specific Config**: `ERRORS_ON_UNMATCHED_IO`, `IO_PIN_H_EXTENSION`, `IO_PIN_H_LAYER`, `IO_PIN_H_LENGTH`, `IO_PIN_H_THICKNESS_MULT`, `IO_PIN_ORDER_CFG`, `IO_PIN_V_EXTENSION`, `IO_PIN_V_LAYER`, `IO_PIN_V_LENGTH`, `IO_PIN_V_THICKNESS_MULT`

##### Flow Python Integration
```python
# Add this step to your flow's sequential Steps list:
class CustomRun(SequentialFlow):
    Steps = [
        ...  # prior steps
        Odb.CustomIOPlacement,
        ...  # subsequent steps
    ]
```
##### Configuration Example
```python
# Configure behavior in the flow's config dict:
config = {
    "IO_PIN_H_LAYER": None,  # (str) The metal layer on which to place horizontally-aligned (long side parallel wi...
    "IO_PIN_V_LAYER": None,  # (str) The metal layer on which to place vertically-aligned (long side perpendicular...
    "IO_PIN_V_EXTENSION": 0,  # (decimal.Decimal) Extends the vertical io pins outside of the die by the specified units.
    "IO_PIN_H_EXTENSION": 0,  # (decimal.Decimal) Extends the horizontal io pins outside of the die by the specified units.
    "IO_PIN_V_THICKNESS_MULT": 2,  # (decimal.Decimal) A multiplier for vertical pin thickness. Base thickness is the pins layer min...
    "IO_PIN_H_THICKNESS_MULT": 2,  # (decimal.Decimal) A multiplier for horizontal pin thickness. Base thickness is the pins layer m...
    "IO_PIN_V_LENGTH": None,  # (Optional[decimal.Decimal]) The length of the pins with a north or south orientation. If unspecified by a...
    "IO_PIN_H_LENGTH": None,  # (Optional[decimal.Decimal]) The length of the pins with an east or west orientation. If unspecified by a ...
    "IO_PIN_ORDER_CFG": None,  # (Optional[librelane.common.types.Path]) Path to a custom pin configuration file.
    "ERRORS_ON_UNMATCHED_IO": 'unmatched_design',  # (Literal['none', 'unmatched_design', 'unmatched_cfg', 'both']) Controls whether to emit an error in: no situation, when pins exist in the de...
}
```

#### `Odb.DiodesOnPorts`
Unconditionally inserts diodes on design ports diodes on ports,
to mitigate the `antenna effect <https://en.wikipedia.org/wiki/Antenna_effect>`_.

Useful for hardening macros, where ports may get long wires that are
unaccounted for when hardening a top-level chip.

The placement is legalized by performing detailed placement and global
routing after inserting the diodes.

Prior to beta 16, this step did not legalize its placement: if you would
like to retain the old behavior without legalization, try
``Odb.PortDiodePlacement``.

* **Inputs**: `odb` (OpenDB Database, `.odb`)
* **Outputs**: `nl` (Verilog Netlist, `.nl.v`), `pnl` (Powered Verilog Netlist, `.pnl.v`), `def` (Design Exchange Format, `.def`), `odb` (OpenDB Database, `.odb`), `sdc` (Design Constraints, `.sdc`)
* **Precursor Steps (Dependencies)**:
* Requires `odb` (OpenDB Database, `.odb`), produced by: `OpenROAD.Floorplan`, `Odb.ApplyDEFTemplate` (or other steps in Odb, OpenROAD)
* **Relevant Step-Specific Config**: `CLOCK_WIRE_RC_LAYERS`, `DEDUPLICATE_CORNERS`, `DIODE_ON_PORTS`, `DIODE_PADDING`, `DPL_CELL_PADDING`, `GPL_CELL_PADDING`, `GRT_ADJUSTMENT`, `GRT_ALLOW_CONGESTION`, `GRT_ANTENNA_REPAIR_DIODE_ONLY`, `GRT_ANTENNA_REPAIR_ITERS`, `GRT_ANTENNA_REPAIR_JUMPER_ONLY`, `GRT_ANTENNA_REPAIR_MARGIN`, `GRT_LAYER_ADJUSTMENTS`, `GRT_MACRO_EXTENSION`, `GRT_OVERFLOW_ITERS`, `LAYERS_RC`, `PDN_CONNECT_MACROS_TO_GRID`, `PDN_ENABLE_GLOBAL_CONNECTIONS`, `PDN_MACRO_CONNECTIONS`, `PL_MAX_DISPLACEMENT_X`, `PL_MAX_DISPLACEMENT_Y`, `PL_OPTIMIZE_MIRRORING`, `PNR_CORNERS`, `PNR_SDC_FILE`, `RT_CLOCK_MAX_LAYER`, `RT_CLOCK_MIN_LAYER`, `SET_RC_VERBOSE`, `SIGNAL_WIRE_RC_LAYERS`, `STA_EXTRA_CORNER_TCL_FILE`, `VIAS_R`

##### Flow Python Integration
```python
# Add this step to your flow's sequential Steps list:
class CustomRun(SequentialFlow):
    Steps = [
        ...  # prior steps
        Odb.DiodesOnPorts,
        ...  # subsequent steps
    ]
```
##### Configuration Example
```python
# Configure behavior in the flow's config dict:
config = {
    "DIODE_ON_PORTS": 'none',  # (Literal['none', 'in', 'out', 'both']) Always insert diodes on ports with the specified polarities.
    "GPL_CELL_PADDING": None,  # (int) Cell padding value (in sites) for global placement. Used by this step only to...
    "PNR_CORNERS": None,  # (Optional[List[str]]) A list of fully-qualified IPVT corners to use during PnR. If unspecified, the...
    "SET_RC_VERBOSE": False,  # (bool) If set to true, set_rc commands are echoed. Quite noisy, but may be useful fo...
    "LAYERS_RC": None,  # (Optional[Dict[str, Dict[str, Dict[str, decimal.Decimal]]]]) Used during PNR steps, Specific custom resistance and capacitance values for ...
    "VIAS_R": None,  # (Optional[Dict[str, Dict[str, Dict[str, decimal.Decimal]]]]) Used during PNR steps, Specific custom resistance values for via layers. For ...
    "SIGNAL_WIRE_RC_LAYERS": None,  # (Optional[List[str]]) Sets estimated signal wire RC values to the average of these layers'. If you ...
    "CLOCK_WIRE_RC_LAYERS": None,  # (Optional[List[str]]) Sets estimated clock wire RC values to the average of these layers'. If you p...
    "PDN_CONNECT_MACROS_TO_GRID": True,  # (bool) Enables the connection of macros to the top level power grid.
    "PDN_MACRO_CONNECTIONS": None,  # (Optional[List[str]]) Specifies explicit power connections of internal macros to the top level powe...
    "PDN_ENABLE_GLOBAL_CONNECTIONS": True,  # (bool) Enables the creation of global connections in PDN generation.
    "PNR_SDC_FILE": None,  # (Optional[librelane.common.types.Path]) Specifies the SDC file used during all implementation (PnR) steps
    "STA_EXTRA_CORNER_TCL_FILE": None,  # (Optional[librelane.common.types.Path]) Experimental: specifies a additional configuration .tcl file to be called dur...
    "DEDUPLICATE_CORNERS": False,  # (bool) Cull duplicate IPVT corners during PNR, i.e. corners that share the same set ...
    "PL_OPTIMIZE_MIRRORING": True,  # (bool) Specifies whether or not to run an optimize_mirroring pass whenever detailed ...
    "PL_MAX_DISPLACEMENT_X": 500,  # (int) Specifies how far an instance can be moved along the X-axis when finding a si...
    "PL_MAX_DISPLACEMENT_Y": 100,  # (int) Specifies how far an instance can be moved along the Y-axis when finding a si...
    "DPL_CELL_PADDING": None,  # (int) Cell padding value (in sites) for detailed placement. The number will be inte...
    "RT_CLOCK_MIN_LAYER": None,  # (Optional[str]) The name of lowest layer to be used in routing the clock net.
    "RT_CLOCK_MAX_LAYER": None,  # (Optional[str]) The name of highest layer to be used in routing the clock net.
    "GRT_ADJUSTMENT": 0.3,  # (decimal.Decimal) Reduction in the routing capacity of the edges between the cells in the globa...
    "GRT_MACRO_EXTENSION": 0,  # (int) Sets the number of GCells added to the blockages boundaries from macros. A GC...
    "GRT_LAYER_ADJUSTMENTS": None,  # (List[decimal.Decimal]) Layer-specific reductions in the routing capacity of the edges between the ce...
    "DIODE_PADDING": None,  # (Optional[int]) Diode cell padding; increases the width of diode cells during placement checks..
    "GRT_ALLOW_CONGESTION": False,  # (bool) Allow congestion during global routing
    "GRT_ANTENNA_REPAIR_ITERS": 3,  # (int) The maximum number of iterations for global antenna repairs.
    "GRT_OVERFLOW_ITERS": 50,  # (int) The maximum number of iterations waiting for the overflow to reach the desire...
    "GRT_ANTENNA_REPAIR_MARGIN": 10,  # (int) The margin to over fix antenna violations.
    "GRT_ANTENNA_REPAIR_JUMPER_ONLY": False,  # (bool) Only use jumpers to fix antenna violations. Cannot be used in conjunction wit...
    "GRT_ANTENNA_REPAIR_DIODE_ONLY": False,  # (bool) Only use antenna diodes to fix antenna violations. Cannot be used in conjunct...
}
```

#### `Odb.FuzzyDiodePlacement`
Runs a custom diode placement script to mitigate the `antenna effect <https://en.wikipedia.org/wiki/Antenna_effect>`_.

This script uses the `Manhattan length <https://en.wikipedia.org/wiki/Manhattan_distance>`_
of a (non-existent) wire at the global placement stage, and places diodes
if they exceed a certain threshold. This, however, requires some padding:
`GPL_CELL_PADDING` and `DPL_CELL_PADDING` must be higher than 0 for this
script to work reliably.

The placement is *not* legalized.

The original script was written by `Sylvain "tnt" Munaut <https://github.com/smunaut>`_.

* **Inputs**: `odb` (OpenDB Database, `.odb`)
* **Outputs**: `odb` (OpenDB Database, `.odb`), `def` (Design Exchange Format, `.def`)
* **Precursor Steps (Dependencies)**:
* Requires `odb` (OpenDB Database, `.odb`), produced by: `OpenROAD.Floorplan`, `Odb.ApplyDEFTemplate` (or other steps in Odb, OpenROAD)
* **Relevant Step-Specific Config**: `GPL_CELL_PADDING`, `HEURISTIC_ANTENNA_THRESHOLD`

##### Flow Python Integration
```python
# Add this step to your flow's sequential Steps list:
class CustomRun(SequentialFlow):
    Steps = [
        ...  # prior steps
        Odb.FuzzyDiodePlacement,
        ...  # subsequent steps
    ]
```
##### Configuration Example
```python
# Configure behavior in the flow's config dict:
config = {
    "HEURISTIC_ANTENNA_THRESHOLD": None,  # (Optional[decimal.Decimal]) A Manhattan distance above which a diode is recommended to be inserted by the...
    "GPL_CELL_PADDING": None,  # (int) Cell padding value (in sites) for global placement. Used by this step only to...
}
```

#### `Odb.HeuristicDiodeInsertion`
Runs a custom diode insertion routine to mitigate the `antenna effect <https://en.wikipedia.org/wiki/Antenna_effect>`_.

This script uses the `Manhattan length <https://en.wikipedia.org/wiki/Manhattan_distance>`_
of a (non-existent) wire at the global placement stage, and places diodes
if they exceed a certain threshold. This, however, requires some padding:
`GPL_CELL_PADDING` and `DPL_CELL_PADDING` must be higher than 0 for this
script to work reliably.

The placement is then legalized by performing detailed placement and global
routing after inserting the diodes.

The original script was written by `Sylvain "tnt" Munaut <https://github.com/smunaut>`_.

Prior to beta 16, this step did not legalize its placement: if you would
like to retain the old behavior without legalization, try
``Odb.FuzzyDiodePlacement``.

* **Inputs**: `odb` (OpenDB Database, `.odb`)
* **Outputs**: `nl` (Verilog Netlist, `.nl.v`), `pnl` (Powered Verilog Netlist, `.pnl.v`), `def` (Design Exchange Format, `.def`), `odb` (OpenDB Database, `.odb`), `sdc` (Design Constraints, `.sdc`)
* **Precursor Steps (Dependencies)**:
* Requires `odb` (OpenDB Database, `.odb`), produced by: `OpenROAD.Floorplan`, `Odb.ApplyDEFTemplate` (or other steps in Odb, OpenROAD)
* **Relevant Step-Specific Config**: `CLOCK_WIRE_RC_LAYERS`, `DEDUPLICATE_CORNERS`, `DIODE_PADDING`, `DPL_CELL_PADDING`, `GPL_CELL_PADDING`, `GRT_ADJUSTMENT`, `GRT_ALLOW_CONGESTION`, `GRT_ANTENNA_REPAIR_DIODE_ONLY`, `GRT_ANTENNA_REPAIR_ITERS`, `GRT_ANTENNA_REPAIR_JUMPER_ONLY`, `GRT_ANTENNA_REPAIR_MARGIN`, `GRT_LAYER_ADJUSTMENTS`, `GRT_MACRO_EXTENSION`, `GRT_OVERFLOW_ITERS`, `HEURISTIC_ANTENNA_THRESHOLD`, `LAYERS_RC`, `PDN_CONNECT_MACROS_TO_GRID`, `PDN_ENABLE_GLOBAL_CONNECTIONS`, `PDN_MACRO_CONNECTIONS`, `PL_MAX_DISPLACEMENT_X`, `PL_MAX_DISPLACEMENT_Y`, `PL_OPTIMIZE_MIRRORING`, `PNR_CORNERS`, `PNR_SDC_FILE`, `RT_CLOCK_MAX_LAYER`, `RT_CLOCK_MIN_LAYER`, `SET_RC_VERBOSE`, `SIGNAL_WIRE_RC_LAYERS`, `STA_EXTRA_CORNER_TCL_FILE`, `VIAS_R`

##### Flow Python Integration
```python
# Add this step to your flow's sequential Steps list:
class CustomRun(SequentialFlow):
    Steps = [
        ...  # prior steps
        Odb.HeuristicDiodeInsertion,
        ...  # subsequent steps
    ]
```
##### Configuration Example
```python
# Configure behavior in the flow's config dict:
config = {
    "HEURISTIC_ANTENNA_THRESHOLD": None,  # (Optional[decimal.Decimal]) A Manhattan distance above which a diode is recommended to be inserted by the...
    "GPL_CELL_PADDING": None,  # (int) Cell padding value (in sites) for global placement. Used by this step only to...
    "PNR_CORNERS": None,  # (Optional[List[str]]) A list of fully-qualified IPVT corners to use during PnR. If unspecified, the...
    "SET_RC_VERBOSE": False,  # (bool) If set to true, set_rc commands are echoed. Quite noisy, but may be useful fo...
    "LAYERS_RC": None,  # (Optional[Dict[str, Dict[str, Dict[str, decimal.Decimal]]]]) Used during PNR steps, Specific custom resistance and capacitance values for ...
    "VIAS_R": None,  # (Optional[Dict[str, Dict[str, Dict[str, decimal.Decimal]]]]) Used during PNR steps, Specific custom resistance values for via layers. For ...
    "SIGNAL_WIRE_RC_LAYERS": None,  # (Optional[List[str]]) Sets estimated signal wire RC values to the average of these layers'. If you ...
    "CLOCK_WIRE_RC_LAYERS": None,  # (Optional[List[str]]) Sets estimated clock wire RC values to the average of these layers'. If you p...
    "PDN_CONNECT_MACROS_TO_GRID": True,  # (bool) Enables the connection of macros to the top level power grid.
    "PDN_MACRO_CONNECTIONS": None,  # (Optional[List[str]]) Specifies explicit power connections of internal macros to the top level powe...
    "PDN_ENABLE_GLOBAL_CONNECTIONS": True,  # (bool) Enables the creation of global connections in PDN generation.
    "PNR_SDC_FILE": None,  # (Optional[librelane.common.types.Path]) Specifies the SDC file used during all implementation (PnR) steps
    "STA_EXTRA_CORNER_TCL_FILE": None,  # (Optional[librelane.common.types.Path]) Experimental: specifies a additional configuration .tcl file to be called dur...
    "DEDUPLICATE_CORNERS": False,  # (bool) Cull duplicate IPVT corners during PNR, i.e. corners that share the same set ...
    "PL_OPTIMIZE_MIRRORING": True,  # (bool) Specifies whether or not to run an optimize_mirroring pass whenever detailed ...
    "PL_MAX_DISPLACEMENT_X": 500,  # (int) Specifies how far an instance can be moved along the X-axis when finding a si...
    "PL_MAX_DISPLACEMENT_Y": 100,  # (int) Specifies how far an instance can be moved along the Y-axis when finding a si...
    "DPL_CELL_PADDING": None,  # (int) Cell padding value (in sites) for detailed placement. The number will be inte...
    "RT_CLOCK_MIN_LAYER": None,  # (Optional[str]) The name of lowest layer to be used in routing the clock net.
    "RT_CLOCK_MAX_LAYER": None,  # (Optional[str]) The name of highest layer to be used in routing the clock net.
    "GRT_ADJUSTMENT": 0.3,  # (decimal.Decimal) Reduction in the routing capacity of the edges between the cells in the globa...
    "GRT_MACRO_EXTENSION": 0,  # (int) Sets the number of GCells added to the blockages boundaries from macros. A GC...
    "GRT_LAYER_ADJUSTMENTS": None,  # (List[decimal.Decimal]) Layer-specific reductions in the routing capacity of the edges between the ce...
    "DIODE_PADDING": None,  # (Optional[int]) Diode cell padding; increases the width of diode cells during placement checks..
    "GRT_ALLOW_CONGESTION": False,  # (bool) Allow congestion during global routing
    "GRT_ANTENNA_REPAIR_ITERS": 3,  # (int) The maximum number of iterations for global antenna repairs.
    "GRT_OVERFLOW_ITERS": 50,  # (int) The maximum number of iterations waiting for the overflow to reach the desire...
    "GRT_ANTENNA_REPAIR_MARGIN": 10,  # (int) The margin to over fix antenna violations.
    "GRT_ANTENNA_REPAIR_JUMPER_ONLY": False,  # (bool) Only use jumpers to fix antenna violations. Cannot be used in conjunction wit...
    "GRT_ANTENNA_REPAIR_DIODE_ONLY": False,  # (bool) Only use antenna diodes to fix antenna violations. Cannot be used in conjunct...
}
```

#### `Odb.InsertECOBuffers`
Experimental step to insert ECO buffers on either drivers or sinks after
global or detailed routing. The placement is legalized and global routing is
incrementally re-run for affected nets. Useful for manually fixing some hold
violations.

If run after detailed routing, detailed routing must be re-run as affected
nets that are altered are removed and require re-routing.

INOUT and FEEDTHRU ports are not supported.

* **Inputs**: `odb` (OpenDB Database, `.odb`)
* **Outputs**: `odb` (OpenDB Database, `.odb`), `def` (Design Exchange Format, `.def`)
* **Precursor Steps (Dependencies)**:
* Requires `odb` (OpenDB Database, `.odb`), produced by: `OpenROAD.Floorplan`, `Odb.ApplyDEFTemplate` (or other steps in Odb, OpenROAD)
* **Relevant Step-Specific Config**: `DIODE_PADDING`, `DPL_CELL_PADDING`, `GRT_ADJUSTMENT`, `GRT_ALLOW_CONGESTION`, `GRT_ANTENNA_REPAIR_DIODE_ONLY`, `GRT_ANTENNA_REPAIR_ITERS`, `GRT_ANTENNA_REPAIR_JUMPER_ONLY`, `GRT_ANTENNA_REPAIR_MARGIN`, `GRT_LAYER_ADJUSTMENTS`, `GRT_MACRO_EXTENSION`, `GRT_OVERFLOW_ITERS`, `INSERT_ECO_BUFFERS`, `PL_MAX_DISPLACEMENT_X`, `PL_MAX_DISPLACEMENT_Y`, `PL_OPTIMIZE_MIRRORING`, `RT_CLOCK_MAX_LAYER`, `RT_CLOCK_MIN_LAYER`

##### Flow Python Integration
```python
# Add this step to your flow's sequential Steps list:
class CustomRun(SequentialFlow):
    Steps = [
        ...  # prior steps
        Odb.InsertECOBuffers,
        ...  # subsequent steps
    ]
```
##### Configuration Example
```python
# Configure behavior in the flow's config dict:
config = {
    "PL_OPTIMIZE_MIRRORING": True,  # (bool) Specifies whether or not to run an optimize_mirroring pass whenever detailed ...
    "PL_MAX_DISPLACEMENT_X": 500,  # (int) Specifies how far an instance can be moved along the X-axis when finding a si...
    "PL_MAX_DISPLACEMENT_Y": 100,  # (int) Specifies how far an instance can be moved along the Y-axis when finding a si...
    "DPL_CELL_PADDING": None,  # (int) Cell padding value (in sites) for detailed placement. The number will be inte...
    "RT_CLOCK_MIN_LAYER": None,  # (Optional[str]) The name of lowest layer to be used in routing the clock net.
    "RT_CLOCK_MAX_LAYER": None,  # (Optional[str]) The name of highest layer to be used in routing the clock net.
    "GRT_ADJUSTMENT": 0.3,  # (decimal.Decimal) Reduction in the routing capacity of the edges between the cells in the globa...
    "GRT_MACRO_EXTENSION": 0,  # (int) Sets the number of GCells added to the blockages boundaries from macros. A GC...
    "GRT_LAYER_ADJUSTMENTS": None,  # (List[decimal.Decimal]) Layer-specific reductions in the routing capacity of the edges between the ce...
    "DIODE_PADDING": None,  # (Optional[int]) Diode cell padding; increases the width of diode cells during placement checks..
    "GRT_ALLOW_CONGESTION": False,  # (bool) Allow congestion during global routing
    "GRT_ANTENNA_REPAIR_ITERS": 3,  # (int) The maximum number of iterations for global antenna repairs.
    "GRT_OVERFLOW_ITERS": 50,  # (int) The maximum number of iterations waiting for the overflow to reach the desire...
    "GRT_ANTENNA_REPAIR_MARGIN": 10,  # (int) The margin to over fix antenna violations.
    "GRT_ANTENNA_REPAIR_JUMPER_ONLY": False,  # (bool) Only use jumpers to fix antenna violations. Cannot be used in conjunction wit...
    "GRT_ANTENNA_REPAIR_DIODE_ONLY": False,  # (bool) Only use antenna diodes to fix antenna violations. Cannot be used in conjunct...
    "INSERT_ECO_BUFFERS": None,  # (Optional[List[librelane.steps.odb.ECOBuffer]]) List of buffers to insert
}
```

#### `Odb.InsertECODiodes`
Experimental step to create and attach ECO diodes to the nets of sinks after
global or detailed routing. The placement is legalized and global routing is
incrementally re-run for affected nets. Useful for manually fixing some
antenna violations.

If run after detailed routing, detailed routing must be re-run as affected
nets that are altered are removed and require re-routing.

* **Inputs**: `odb` (OpenDB Database, `.odb`)
* **Outputs**: `odb` (OpenDB Database, `.odb`), `def` (Design Exchange Format, `.def`)
* **Precursor Steps (Dependencies)**:
* Requires `odb` (OpenDB Database, `.odb`), produced by: `OpenROAD.Floorplan`, `Odb.ApplyDEFTemplate` (or other steps in Odb, OpenROAD)
* **Relevant Step-Specific Config**: `DIODE_PADDING`, `DPL_CELL_PADDING`, `GRT_ADJUSTMENT`, `GRT_ALLOW_CONGESTION`, `GRT_ANTENNA_REPAIR_DIODE_ONLY`, `GRT_ANTENNA_REPAIR_ITERS`, `GRT_ANTENNA_REPAIR_JUMPER_ONLY`, `GRT_ANTENNA_REPAIR_MARGIN`, `GRT_LAYER_ADJUSTMENTS`, `GRT_MACRO_EXTENSION`, `GRT_OVERFLOW_ITERS`, `INSERT_ECO_DIODES`, `PL_MAX_DISPLACEMENT_X`, `PL_MAX_DISPLACEMENT_Y`, `PL_OPTIMIZE_MIRRORING`, `RT_CLOCK_MAX_LAYER`, `RT_CLOCK_MIN_LAYER`

##### Flow Python Integration
```python
# Add this step to your flow's sequential Steps list:
class CustomRun(SequentialFlow):
    Steps = [
        ...  # prior steps
        Odb.InsertECODiodes,
        ...  # subsequent steps
    ]
```
##### Configuration Example
```python
# Configure behavior in the flow's config dict:
config = {
    "RT_CLOCK_MIN_LAYER": None,  # (Optional[str]) The name of lowest layer to be used in routing the clock net.
    "RT_CLOCK_MAX_LAYER": None,  # (Optional[str]) The name of highest layer to be used in routing the clock net.
    "GRT_ADJUSTMENT": 0.3,  # (decimal.Decimal) Reduction in the routing capacity of the edges between the cells in the globa...
    "GRT_MACRO_EXTENSION": 0,  # (int) Sets the number of GCells added to the blockages boundaries from macros. A GC...
    "GRT_LAYER_ADJUSTMENTS": None,  # (List[decimal.Decimal]) Layer-specific reductions in the routing capacity of the edges between the ce...
    "DIODE_PADDING": None,  # (Optional[int]) Diode cell padding; increases the width of diode cells during placement checks..
    "GRT_ALLOW_CONGESTION": False,  # (bool) Allow congestion during global routing
    "GRT_ANTENNA_REPAIR_ITERS": 3,  # (int) The maximum number of iterations for global antenna repairs.
    "GRT_OVERFLOW_ITERS": 50,  # (int) The maximum number of iterations waiting for the overflow to reach the desire...
    "GRT_ANTENNA_REPAIR_MARGIN": 10,  # (int) The margin to over fix antenna violations.
    "GRT_ANTENNA_REPAIR_JUMPER_ONLY": False,  # (bool) Only use jumpers to fix antenna violations. Cannot be used in conjunction wit...
    "GRT_ANTENNA_REPAIR_DIODE_ONLY": False,  # (bool) Only use antenna diodes to fix antenna violations. Cannot be used in conjunct...
    "PL_OPTIMIZE_MIRRORING": True,  # (bool) Specifies whether or not to run an optimize_mirroring pass whenever detailed ...
    "PL_MAX_DISPLACEMENT_X": 500,  # (int) Specifies how far an instance can be moved along the X-axis when finding a si...
    "PL_MAX_DISPLACEMENT_Y": 100,  # (int) Specifies how far an instance can be moved along the Y-axis when finding a si...
    "DPL_CELL_PADDING": None,  # (int) Cell padding value (in sites) for detailed placement. The number will be inte...
    "INSERT_ECO_DIODES": None,  # (Optional[List[librelane.steps.odb.ECODiode]]) List of sinks to insert diodes for.
}
```

#### `Odb.ManualGlobalPlacement`
This is an step to override the placement of one or more instances at
user-specified locations.

Alternatively, if this is a custom design with a few cells, this can be used
in place of the global placement entirely.

* **Inputs**: `odb` (OpenDB Database, `.odb`)
* **Outputs**: `odb` (OpenDB Database, `.odb`), `def` (Design Exchange Format, `.def`)
* **Precursor Steps (Dependencies)**:
* Requires `odb` (OpenDB Database, `.odb`), produced by: `OpenROAD.Floorplan`, `Odb.ApplyDEFTemplate` (or other steps in Odb, OpenROAD)
* **Relevant Step-Specific Config**: `MANUAL_GLOBAL_PLACEMENTS`

##### Flow Python Integration
```python
# Add this step to your flow's sequential Steps list:
class CustomRun(SequentialFlow):
    Steps = [
        ...  # prior steps
        Odb.ManualGlobalPlacement,
        ...  # subsequent steps
    ]
```
##### Configuration Example
```python
# Configure behavior in the flow's config dict:
config = {
    "MANUAL_GLOBAL_PLACEMENTS": None,  # (Optional[Dict[str, librelane.config.variable.Instance]]) A dictionary of instances to their global (non-legalized and unfixed) placeme...
}
```

#### `Odb.ManualMacroPlacement`
Performs macro placement using a simple configuration file. The file is
defined as a line-break delimited list of instances and positions, in the
format ``instance_name X_pos Y_pos Orientation``.

If no macro instances are configured, this step is skipped.

* **Inputs**: `odb` (OpenDB Database, `.odb`)
* **Outputs**: `odb` (OpenDB Database, `.odb`), `def` (Design Exchange Format, `.def`)
* **Precursor Steps (Dependencies)**:
* Requires `odb` (OpenDB Database, `.odb`), produced by: `OpenROAD.Floorplan`, `Odb.ApplyDEFTemplate` (or other steps in Odb, OpenROAD)
* **Relevant Step-Specific Config**: `MACRO_PLACEMENT_CFG`

##### Flow Python Integration
```python
# Add this step to your flow's sequential Steps list:
class CustomRun(SequentialFlow):
    Steps = [
        ...  # prior steps
        Odb.ManualMacroPlacement,
        ...  # subsequent steps
    ]
```
##### Configuration Example
```python
# Configure behavior in the flow's config dict:
config = {
    "MACRO_PLACEMENT_CFG": None,  # (Optional[librelane.common.types.Path]) Path to an optional override for instance placement instead of the `MACROS` o...
}
```

#### `Odb.PortDiodePlacement`
Unconditionally inserts diodes on design ports diodes on ports,
to mitigate the `antenna effect <https://en.wikipedia.org/wiki/Antenna_effect>`_.

Useful for hardening macros, where ports may get long wires that are
unaccounted for when hardening a top-level chip.

The placement is **not legalized**.

* **Inputs**: `odb` (OpenDB Database, `.odb`)
* **Outputs**: `odb` (OpenDB Database, `.odb`), `def` (Design Exchange Format, `.def`)
* **Precursor Steps (Dependencies)**:
* Requires `odb` (OpenDB Database, `.odb`), produced by: `OpenROAD.Floorplan`, `Odb.ApplyDEFTemplate` (or other steps in Odb, OpenROAD)
* **Relevant Step-Specific Config**: `DIODE_ON_PORTS`, `GPL_CELL_PADDING`

##### Flow Python Integration
```python
# Add this step to your flow's sequential Steps list:
class CustomRun(SequentialFlow):
    Steps = [
        ...  # prior steps
        Odb.PortDiodePlacement,
        ...  # subsequent steps
    ]
```
##### Configuration Example
```python
# Configure behavior in the flow's config dict:
config = {
    "DIODE_ON_PORTS": 'none',  # (Literal['none', 'in', 'out', 'both']) Always insert diodes on ports with the specified polarities.
    "GPL_CELL_PADDING": None,  # (int) Cell padding value (in sites) for global placement. Used by this step only to...
}
```

#### `Odb.RemovePDNObstructions`
Removes any PDN obstructions previously placed by
<#Odb.RemovePDNObstructions>`_.

* **Inputs**: `odb` (OpenDB Database, `.odb`)
* **Outputs**: `odb` (OpenDB Database, `.odb`), `def` (Design Exchange Format, `.def`)
* **Precursor Steps (Dependencies)**:
* Requires `odb` (OpenDB Database, `.odb`), produced by: `OpenROAD.Floorplan`, `Odb.ApplyDEFTemplate` (or other steps in Odb, OpenROAD)
* **Relevant Step-Specific Config**: `PDN_OBSTRUCTIONS`

##### Flow Python Integration
```python
# Add this step to your flow's sequential Steps list:
class CustomRun(SequentialFlow):
    Steps = [
        ...  # prior steps
        Odb.RemovePDNObstructions,
        ...  # subsequent steps
    ]
```
##### Configuration Example
```python
# Configure behavior in the flow's config dict:
config = {
    "PDN_OBSTRUCTIONS": None,  # (Optional[List[Tuple[str, decimal.Decimal, decimal.Decimal, decimal.Decimal, decimal.Decimal]]]) Add routing obstructions to the design before PDN stage. If set to `None`, th...
}
```

#### `Odb.RemoveRoutingObstructions`
Removes any routing obstructions previously placed by
<#Odb.AddRoutingObstructions>`_.

* **Inputs**: `odb` (OpenDB Database, `.odb`)
* **Outputs**: `odb` (OpenDB Database, `.odb`), `def` (Design Exchange Format, `.def`)
* **Precursor Steps (Dependencies)**:
* Requires `odb` (OpenDB Database, `.odb`), produced by: `OpenROAD.Floorplan`, `Odb.ApplyDEFTemplate` (or other steps in Odb, OpenROAD)
* **Relevant Step-Specific Config**: `ROUTING_OBSTRUCTIONS`

##### Flow Python Integration
```python
# Add this step to your flow's sequential Steps list:
class CustomRun(SequentialFlow):
    Steps = [
        ...  # prior steps
        Odb.RemoveRoutingObstructions,
        ...  # subsequent steps
    ]
```
##### Configuration Example
```python
# Configure behavior in the flow's config dict:
config = {
    "ROUTING_OBSTRUCTIONS": None,  # (Optional[List[Tuple[str, decimal.Decimal, decimal.Decimal, decimal.Decimal, decimal.Decimal]]]) Add routing obstructions to the design. If set to `None`, this step is skippe...
}
```

#### `Odb.ReportDisconnectedPins`
Creates a table of disconnected pins in the design, updating metrics as
appropriate.

Disconnected pins may be marked "critical" if they are very likely to
result in a dead design. We determine if a pin is critical as follows:

* For the top-level macro: for these four kinds of pins: inputs, outputs,
power inouts, and ground inouts, at least one of each kind must be
connected or else all pins of a certain kind are counted as critical
disconnected pins.
* For instances:
* Any unconnected input is a critical disconnected pin.
* If there isn't at least one output connected, all disconnected
outputs are critical disconnected pins.
* Any disconnected power inout pins are critical disconnected pins.

The metrics ``design__disconnected_pin__count`` and
``design__critical_disconnected_pin__count`` is updated. It is recommended
to use the checker ``Checker.DisconnectedPins`` to check that there are
no critical disconnected pins.

* **Inputs**: `odb` (OpenDB Database, `.odb`)
* **Outputs**: `odb` (OpenDB Database, `.odb`), `def` (Design Exchange Format, `.def`)
* **Precursor Steps (Dependencies)**:
* Requires `odb` (OpenDB Database, `.odb`), produced by: `OpenROAD.Floorplan`, `Odb.ApplyDEFTemplate` (or other steps in Odb, OpenROAD)
* **Relevant Step-Specific Config**: `IGNORE_DISCONNECTED_MODULES`

##### Flow Python Integration
```python
# Add this step to your flow's sequential Steps list:
class CustomRun(SequentialFlow):
    Steps = [
        ...  # prior steps
        Odb.ReportDisconnectedPins,
        ...  # subsequent steps
    ]
```
##### Configuration Example
```python
# Configure behavior in the flow's config dict:
config = {
    "IGNORE_DISCONNECTED_MODULES": None,  # (Optional[List[str]]) Modules (or cells) to ignore when checking for disconnected pins.
}
```

#### `Odb.ReportWireLength`
Outputs a CSV of long wires, printed by length. Useful as a design aid to
detect when one wire is connected to too many things.

* **Inputs**: `odb` (OpenDB Database, `.odb`)
* **Outputs**: None
* **Precursor Steps (Dependencies)**:
* Requires `odb` (OpenDB Database, `.odb`), produced by: `OpenROAD.Floorplan`, `Odb.ApplyDEFTemplate` (or other steps in Odb, OpenROAD)

##### Flow Python Integration
```python
# Add this step to your flow's sequential Steps list:
class CustomRun(SequentialFlow):
    Steps = [
        ...  # prior steps
        Odb.ReportWireLength,
        ...  # subsequent steps
    ]
```

#### `Odb.SetPowerConnections`
Uses JSON netlist and module information in Odb to add global power
connections for macros at the top level of a design.

If the JSON netlist is hierarchical (e.g. by using a keep hierarchy
attribute) this Step emits a warning and does not attempt to connect any
macros instantiated within submodules.

* **Inputs**: `json_h` (Design JSON Header File, `.h.json`), `odb` (OpenDB Database, `.odb`)
* **Outputs**: `odb` (OpenDB Database, `.odb`), `def` (Design Exchange Format, `.def`)
* **Precursor Steps (Dependencies)**:
* Requires `json_h` (Design JSON Header File, `.h.json`), produced by: `Yosys.JsonHeader`
* Requires `odb` (OpenDB Database, `.odb`), produced by: `OpenROAD.Floorplan`, `Odb.ApplyDEFTemplate` (or other steps in Odb, OpenROAD)

##### Flow Python Integration
```python
# Add this step to your flow's sequential Steps list:
class CustomRun(SequentialFlow):
    Steps = [
        ...  # prior steps
        Odb.SetPowerConnections,
        ...  # subsequent steps
    ]
```

#### `Odb.WriteVerilogHeader`
Writes a Verilog header of the module using information from the generated
PDN, guarded by the value of ``VERILOG_POWER_DEFINE``, and the JSON header.

* **Inputs**: `odb` (OpenDB Database, `.odb`), `json_h` (Design JSON Header File, `.h.json`)
* **Outputs**: `vh` (Verilog Header, `.vh`)
* **Precursor Steps (Dependencies)**:
* Requires `odb` (OpenDB Database, `.odb`), produced by: `OpenROAD.Floorplan`, `Odb.ApplyDEFTemplate` (or other steps in Odb, OpenROAD)
* Requires `json_h` (Design JSON Header File, `.h.json`), produced by: `Yosys.JsonHeader`
* **Relevant Step-Specific Config**: `VERILOG_POWER_DEFINE`

##### Flow Python Integration
```python
# Add this step to your flow's sequential Steps list:
class CustomRun(SequentialFlow):
    Steps = [
        ...  # prior steps
        Odb.WriteVerilogHeader,
        ...  # subsequent steps
    ]
```
##### Configuration Example
```python
# Configure behavior in the flow's config dict:
config = {
    "VERILOG_POWER_DEFINE": 'USE_POWER_PINS',  # (Optional[str]) Specifies the name of the define used to guard power and ground connections i...
}
```

### OpenROAD Steps

#### `OpenROAD.CTS`
Creates a `Clock tree <https://en.wikipedia.org/wiki/Clock_signal#Distribution>`_
for an ODB file with detailed-placed cells, using reasonably accurate resistance
and capacitance estimations. Detailed Placement is then re-performed to
accommodate the new cells.

* **Inputs**: `odb` (OpenDB Database, `.odb`)
* **Outputs**: `odb` (OpenDB Database, `.odb`), `def` (Design Exchange Format, `.def`), `sdc` (Design Constraints, `.sdc`), `nl` (Verilog Netlist, `.nl.v`), `pnl` (Powered Verilog Netlist, `.pnl.v`)
* **Precursor Steps (Dependencies)**:
* Requires `odb` (OpenDB Database, `.odb`), produced by: `OpenROAD.Floorplan`, `Odb.ApplyDEFTemplate` (or other steps in Odb, OpenROAD)
* **Relevant Step-Specific Config**: `CLOCK_WIRE_RC_LAYERS`, `CTS_APPLY_NDR`, `CTS_BALANCE_LEVELS`, `CTS_CLK_BUFFERS`, `CTS_CLK_MAX_WIRE_LENGTH`, `CTS_CORNERS`, `CTS_DELAY_BUFFER_DERATE_PCT`, `CTS_DISABLE_POST_PROCESSING`, `CTS_DISTANCE_BETWEEN_BUFFERS`, `CTS_MACRO_CLUSTERING_MAX_DIAMETER`, `CTS_MACRO_CLUSTERING_SIZE`, `CTS_MAX_CAP`, `CTS_MAX_SLEW`, `CTS_OBSTRUCTION_AWARE`, `CTS_ROOT_BUFFER`, `CTS_SINK_BUFFER_MAX_CAP_DERATE_PCT`, `CTS_SINK_CLUSTERING_ENABLE`, `CTS_SINK_CLUSTERING_MAX_DIAMETER`, `CTS_SINK_CLUSTERING_SIZE`, `DEDUPLICATE_CORNERS`, `DPL_CELL_PADDING`, `LAYERS_RC`, `PDN_CONNECT_MACROS_TO_GRID`, `PDN_ENABLE_GLOBAL_CONNECTIONS`, `PDN_MACRO_CONNECTIONS`, `PL_MAX_DISPLACEMENT_X`, `PL_MAX_DISPLACEMENT_Y`, `PL_OPTIMIZE_MIRRORING`, `PNR_CORNERS`, `PNR_SDC_FILE`, `SET_RC_VERBOSE`, `SIGNAL_WIRE_RC_LAYERS`, `STA_EXTRA_CORNER_TCL_FILE`, `VIAS_R`

##### Flow Python Integration
```python
# Add this step to your flow's sequential Steps list:
class CustomRun(SequentialFlow):
    Steps = [
        ...  # prior steps
        OpenROAD.CTS,
        ...  # subsequent steps
    ]
```
##### Configuration Example
```python
# Configure behavior in the flow's config dict:
config = {
    "PNR_CORNERS": None,  # (Optional[List[str]]) A list of fully-qualified IPVT corners to use during PnR. If unspecified, the...
    "SET_RC_VERBOSE": False,  # (bool) If set to true, set_rc commands are echoed. Quite noisy, but may be useful fo...
    "LAYERS_RC": None,  # (Optional[Dict[str, Dict[str, Dict[str, decimal.Decimal]]]]) Used during PNR steps, Specific custom resistance and capacitance values for ...
    "VIAS_R": None,  # (Optional[Dict[str, Dict[str, Dict[str, decimal.Decimal]]]]) Used during PNR steps, Specific custom resistance values for via layers. For ...
    "SIGNAL_WIRE_RC_LAYERS": None,  # (Optional[List[str]]) Sets estimated signal wire RC values to the average of these layers'. If you ...
    "CLOCK_WIRE_RC_LAYERS": None,  # (Optional[List[str]]) Sets estimated clock wire RC values to the average of these layers'. If you p...
    "PDN_CONNECT_MACROS_TO_GRID": True,  # (bool) Enables the connection of macros to the top level power grid.
    "PDN_MACRO_CONNECTIONS": None,  # (Optional[List[str]]) Specifies explicit power connections of internal macros to the top level powe...
    "PDN_ENABLE_GLOBAL_CONNECTIONS": True,  # (bool) Enables the creation of global connections in PDN generation.
    "PNR_SDC_FILE": None,  # (Optional[librelane.common.types.Path]) Specifies the SDC file used during all implementation (PnR) steps
    "STA_EXTRA_CORNER_TCL_FILE": None,  # (Optional[librelane.common.types.Path]) Experimental: specifies a additional configuration .tcl file to be called dur...
    "DEDUPLICATE_CORNERS": False,  # (bool) Cull duplicate IPVT corners during PNR, i.e. corners that share the same set ...
    "PL_OPTIMIZE_MIRRORING": True,  # (bool) Specifies whether or not to run an optimize_mirroring pass whenever detailed ...
    "PL_MAX_DISPLACEMENT_X": 500,  # (int) Specifies how far an instance can be moved along the X-axis when finding a si...
    "PL_MAX_DISPLACEMENT_Y": 100,  # (int) Specifies how far an instance can be moved along the Y-axis when finding a si...
    "DPL_CELL_PADDING": None,  # (int) Cell padding value (in sites) for detailed placement. The number will be inte...
    "CTS_BALANCE_LEVELS": None,  # (Optional[bool]) Attempts to keep a similar number of levels in the clock tree across non-regi...
    "CTS_SINK_BUFFER_MAX_CAP_DERATE_PCT": None,  # (Optional[decimal.Decimal]) Controls automatic buffer selection. To favor strong(weak) drive strength buf...
    "CTS_DELAY_BUFFER_DERATE_PCT": None,  # (Optional[decimal.Decimal]) This option balances latencies between macro cells and registers by inserting...
    "CTS_OBSTRUCTION_AWARE": None,  # (Optional[bool]) Enables obstruction-aware buffering such that clock buffers are not placed on...
    "CTS_SINK_CLUSTERING_ENABLE": True,  # (bool) Enables pre-clustering of sinks to create one level of sub-tree before buildi...
    "CTS_SINK_CLUSTERING_SIZE": None,  # (Optional[int]) Specifies the maximum number of sinks per cluster.
    "CTS_SINK_CLUSTERING_MAX_DIAMETER": None,  # (Optional[decimal.Decimal]) Specifies the maximum diameter of the sink cluster.
    "CTS_MACRO_CLUSTERING_SIZE": None,  # (Optional[int]) Specifies the maximum number of sinks per cluster for the macro tree.
    "CTS_MACRO_CLUSTERING_MAX_DIAMETER": None,  # (Optional[decimal.Decimal]) Specifies the maximum diameter of the sink cluster for the macro tree.
    "CTS_CLK_MAX_WIRE_LENGTH": 0,  # (decimal.Decimal) Specifies the maximum wire length on the clock net.
    "CTS_DISABLE_POST_PROCESSING": False,  # (bool) Specifies whether or not to disable post cts processing for outlier sinks.
    "CTS_DISTANCE_BETWEEN_BUFFERS": 0,  # (decimal.Decimal) Specifies the distance between buffers when creating the clock tree.
    "CTS_CORNERS": None,  # (Optional[List[str]]) Clock tree synthesis step-specific override for PNR_CORNERS.
    "CTS_ROOT_BUFFER": None,  # (str) Defines the cell inserted at the root of the clock tree. Used in CTS.
    "CTS_CLK_BUFFERS": None,  # (List[str]) Defines the list of clock buffer names or buffer name wildcards to be used in...
    "CTS_MAX_CAP": None,  # (Optional[decimal.Decimal]) Overrides the maximum capacitance CTS characterization will test. If omitted,...
    "CTS_MAX_SLEW": None,  # (Optional[decimal.Decimal]) Overrides the maximum transition time CTS characterization will test. If omit...
    "CTS_APPLY_NDR": 'half',  # (Literal['none', 'root_only', 'half', 'full']) Applies 2X spacing non-default rule to clock nets except leaf-level nets foll...
}
```

#### `OpenROAD.CheckAntennas`
Runs OpenROAD to check if one or more long nets may constitute an
`antenna risk <https://en.wikipedia.org/wiki/Antenna_effect>`_.

The metric ``route__antenna_violation__count`` will be updated with the number of violating nets.

* **Inputs**: `odb` (OpenDB Database, `.odb`)
* **Outputs**: None
* **Precursor Steps (Dependencies)**:
* Requires `odb` (OpenDB Database, `.odb`), produced by: `OpenROAD.Floorplan`, `Odb.ApplyDEFTemplate` (or other steps in Odb, OpenROAD)
* **Relevant Step-Specific Config**: `CLOCK_WIRE_RC_LAYERS`, `DEDUPLICATE_CORNERS`, `LAYERS_RC`, `PDN_CONNECT_MACROS_TO_GRID`, `PDN_ENABLE_GLOBAL_CONNECTIONS`, `PDN_MACRO_CONNECTIONS`, `PNR_CORNERS`, `PNR_SDC_FILE`, `SET_RC_VERBOSE`, `SIGNAL_WIRE_RC_LAYERS`, `STA_EXTRA_CORNER_TCL_FILE`, `VIAS_R`

##### Flow Python Integration
```python
# Add this step to your flow's sequential Steps list:
class CustomRun(SequentialFlow):
    Steps = [
        ...  # prior steps
        OpenROAD.CheckAntennas,
        ...  # subsequent steps
    ]
```
##### Configuration Example
```python
# Configure behavior in the flow's config dict:
config = {
    "PNR_CORNERS": None,  # (Optional[List[str]]) A list of fully-qualified IPVT corners to use during PnR. If unspecified, the...
    "SET_RC_VERBOSE": False,  # (bool) If set to true, set_rc commands are echoed. Quite noisy, but may be useful fo...
    "LAYERS_RC": None,  # (Optional[Dict[str, Dict[str, Dict[str, decimal.Decimal]]]]) Used during PNR steps, Specific custom resistance and capacitance values for ...
    "VIAS_R": None,  # (Optional[Dict[str, Dict[str, Dict[str, decimal.Decimal]]]]) Used during PNR steps, Specific custom resistance values for via layers. For ...
    "SIGNAL_WIRE_RC_LAYERS": None,  # (Optional[List[str]]) Sets estimated signal wire RC values to the average of these layers'. If you ...
    "CLOCK_WIRE_RC_LAYERS": None,  # (Optional[List[str]]) Sets estimated clock wire RC values to the average of these layers'. If you p...
    "PDN_CONNECT_MACROS_TO_GRID": True,  # (bool) Enables the connection of macros to the top level power grid.
    "PDN_MACRO_CONNECTIONS": None,  # (Optional[List[str]]) Specifies explicit power connections of internal macros to the top level powe...
    "PDN_ENABLE_GLOBAL_CONNECTIONS": True,  # (bool) Enables the creation of global connections in PDN generation.
    "PNR_SDC_FILE": None,  # (Optional[librelane.common.types.Path]) Specifies the SDC file used during all implementation (PnR) steps
    "STA_EXTRA_CORNER_TCL_FILE": None,  # (Optional[librelane.common.types.Path]) Experimental: specifies a additional configuration .tcl file to be called dur...
    "DEDUPLICATE_CORNERS": False,  # (bool) Cull duplicate IPVT corners during PNR, i.e. corners that share the same set ...
}
```

#### `OpenROAD.CheckMacroInstances`
Checks if all macro instances declared in the configuration are, in fact,
in the design, emitting an error otherwise.

Nested macros (macros within macros) are supported provided netlist views
are available for the macro.

* **Inputs**: `nl` (Verilog Netlist, `.nl.v`)
* **Outputs**: None
* **Precursor Steps (Dependencies)**:
* Requires `nl` (Verilog Netlist, `.nl.v`), produced by: `Yosys.Synthesis`, `OpenROAD.Floorplan`, `Odb.DiodesOnPorts` (or other steps in Odb, OpenROAD, Yosys)
* **Relevant Step-Specific Config**: `CLOCK_WIRE_RC_LAYERS`, `DEDUPLICATE_CORNERS`, `LAYERS_RC`, `PDN_CONNECT_MACROS_TO_GRID`, `PDN_ENABLE_GLOBAL_CONNECTIONS`, `PDN_MACRO_CONNECTIONS`, `PNR_CORNERS`, `PNR_SDC_FILE`, `SET_RC_VERBOSE`, `SIGNAL_WIRE_RC_LAYERS`, `STA_EXTRA_CORNER_TCL_FILE`, `VIAS_R`

##### Flow Python Integration
```python
# Add this step to your flow's sequential Steps list:
class CustomRun(SequentialFlow):
    Steps = [
        ...  # prior steps
        OpenROAD.CheckMacroInstances,
        ...  # subsequent steps
    ]
```
##### Configuration Example
```python
# Configure behavior in the flow's config dict:
config = {
    "PNR_CORNERS": None,  # (Optional[List[str]]) A list of fully-qualified IPVT corners to use during PnR. If unspecified, the...
    "SET_RC_VERBOSE": False,  # (bool) If set to true, set_rc commands are echoed. Quite noisy, but may be useful fo...
    "LAYERS_RC": None,  # (Optional[Dict[str, Dict[str, Dict[str, decimal.Decimal]]]]) Used during PNR steps, Specific custom resistance and capacitance values for ...
    "VIAS_R": None,  # (Optional[Dict[str, Dict[str, Dict[str, decimal.Decimal]]]]) Used during PNR steps, Specific custom resistance values for via layers. For ...
    "SIGNAL_WIRE_RC_LAYERS": None,  # (Optional[List[str]]) Sets estimated signal wire RC values to the average of these layers'. If you ...
    "CLOCK_WIRE_RC_LAYERS": None,  # (Optional[List[str]]) Sets estimated clock wire RC values to the average of these layers'. If you p...
    "PDN_CONNECT_MACROS_TO_GRID": True,  # (bool) Enables the connection of macros to the top level power grid.
    "PDN_MACRO_CONNECTIONS": None,  # (Optional[List[str]]) Specifies explicit power connections of internal macros to the top level powe...
    "PDN_ENABLE_GLOBAL_CONNECTIONS": True,  # (bool) Enables the creation of global connections in PDN generation.
    "PNR_SDC_FILE": None,  # (Optional[librelane.common.types.Path]) Specifies the SDC file used during all implementation (PnR) steps
    "STA_EXTRA_CORNER_TCL_FILE": None,  # (Optional[librelane.common.types.Path]) Experimental: specifies a additional configuration .tcl file to be called dur...
    "DEDUPLICATE_CORNERS": False,  # (bool) Cull duplicate IPVT corners during PNR, i.e. corners that share the same set ...
}
```

#### `OpenROAD.CheckSDCFiles`
Checks that the two variables used for SDC files by OpenROAD steps,
namely, ``PNR_SDC_FILE`` and ``SIGNOFF_SDC_FILE``, are explicitly set to
valid paths by the users, and emits a warning that the fallback will be
utilized otherwise.

* **Inputs**: None
* **Outputs**: None
* **Precursor Steps (Dependencies)**:
None (typically runs at the start of the flow, or only requires static design files)
* **Relevant Step-Specific Config**: `PNR_SDC_FILE`, `SIGNOFF_SDC_FILE`

##### Flow Python Integration
```python
# Add this step to your flow's sequential Steps list:
class CustomRun(SequentialFlow):
    Steps = [
        ...  # prior steps
        OpenROAD.CheckSDCFiles,
        ...  # subsequent steps
    ]
```
##### Configuration Example
```python
# Configure behavior in the flow's config dict:
config = {
    "PNR_SDC_FILE": None,  # (Optional[librelane.common.types.Path]) Specifies the SDC file used during all implementation (PnR) steps
    "SIGNOFF_SDC_FILE": None,  # (Optional[librelane.common.types.Path]) Specifies the SDC file for STA during signoff
}
```

#### `OpenROAD.CutRows`
Cut floorplan rows with respect to placed macros.

* **Inputs**: `odb` (OpenDB Database, `.odb`)
* **Outputs**: `odb` (OpenDB Database, `.odb`), `def` (Design Exchange Format, `.def`)
* **Precursor Steps (Dependencies)**:
* Requires `odb` (OpenDB Database, `.odb`), produced by: `OpenROAD.Floorplan`, `Odb.ApplyDEFTemplate` (or other steps in Odb, OpenROAD)
* **Relevant Step-Specific Config**: `CLOCK_WIRE_RC_LAYERS`, `DEDUPLICATE_CORNERS`, `FP_MACRO_HORIZONTAL_HALO`, `FP_MACRO_VERTICAL_HALO`, `FP_PRUNE_THRESHOLD`, `LAYERS_RC`, `PDN_CONNECT_MACROS_TO_GRID`, `PDN_ENABLE_GLOBAL_CONNECTIONS`, `PDN_MACRO_CONNECTIONS`, `PNR_CORNERS`, `PNR_SDC_FILE`, `SET_RC_VERBOSE`, `SIGNAL_WIRE_RC_LAYERS`, `STA_EXTRA_CORNER_TCL_FILE`, `VIAS_R`

##### Flow Python Integration
```python
# Add this step to your flow's sequential Steps list:
class CustomRun(SequentialFlow):
    Steps = [
        ...  # prior steps
        OpenROAD.CutRows,
        ...  # subsequent steps
    ]
```
##### Configuration Example
```python
# Configure behavior in the flow's config dict:
config = {
    "PNR_CORNERS": None,  # (Optional[List[str]]) A list of fully-qualified IPVT corners to use during PnR. If unspecified, the...
    "SET_RC_VERBOSE": False,  # (bool) If set to true, set_rc commands are echoed. Quite noisy, but may be useful fo...
    "LAYERS_RC": None,  # (Optional[Dict[str, Dict[str, Dict[str, decimal.Decimal]]]]) Used during PNR steps, Specific custom resistance and capacitance values for ...
    "VIAS_R": None,  # (Optional[Dict[str, Dict[str, Dict[str, decimal.Decimal]]]]) Used during PNR steps, Specific custom resistance values for via layers. For ...
    "SIGNAL_WIRE_RC_LAYERS": None,  # (Optional[List[str]]) Sets estimated signal wire RC values to the average of these layers'. If you ...
    "CLOCK_WIRE_RC_LAYERS": None,  # (Optional[List[str]]) Sets estimated clock wire RC values to the average of these layers'. If you p...
    "PDN_CONNECT_MACROS_TO_GRID": True,  # (bool) Enables the connection of macros to the top level power grid.
    "PDN_MACRO_CONNECTIONS": None,  # (Optional[List[str]]) Specifies explicit power connections of internal macros to the top level powe...
    "PDN_ENABLE_GLOBAL_CONNECTIONS": True,  # (bool) Enables the creation of global connections in PDN generation.
    "PNR_SDC_FILE": None,  # (Optional[librelane.common.types.Path]) Specifies the SDC file used during all implementation (PnR) steps
    "STA_EXTRA_CORNER_TCL_FILE": None,  # (Optional[librelane.common.types.Path]) Experimental: specifies a additional configuration .tcl file to be called dur...
    "DEDUPLICATE_CORNERS": False,  # (bool) Cull duplicate IPVT corners during PNR, i.e. corners that share the same set ...
    "FP_MACRO_HORIZONTAL_HALO": 10,  # (decimal.Decimal) Specify the horizontal halo size around macros.
    "FP_MACRO_VERTICAL_HALO": 10,  # (decimal.Decimal) Specify the vertical halo size around macros.
    "FP_PRUNE_THRESHOLD": None,  # (Optional[decimal.Decimal]) If specified, all rows smaller in width than this value will be removed. This...
}
```

#### `OpenROAD.DEFtoODB`
Converts a DEF view to an ODB view.

Useful if you have a custom step that manipulates the layout outside of
OpenROAD, but you would like to update the OpenROAD database.

* **Inputs**: `def` (Design Exchange Format, `.def`)
* **Outputs**: `odb` (OpenDB Database, `.odb`)
* **Precursor Steps (Dependencies)**:
* Requires `def` (Design Exchange Format, `.def`), produced by: `OpenROAD.Floorplan`, `Odb.ApplyDEFTemplate` (or other steps in Odb, OpenROAD)
* **Relevant Step-Specific Config**: `CLOCK_WIRE_RC_LAYERS`, `DEDUPLICATE_CORNERS`, `LAYERS_RC`, `PDN_CONNECT_MACROS_TO_GRID`, `PDN_ENABLE_GLOBAL_CONNECTIONS`, `PDN_MACRO_CONNECTIONS`, `PNR_CORNERS`, `PNR_SDC_FILE`, `SET_RC_VERBOSE`, `SIGNAL_WIRE_RC_LAYERS`, `STA_EXTRA_CORNER_TCL_FILE`, `VIAS_R`

##### Flow Python Integration
```python
# Add this step to your flow's sequential Steps list:
class CustomRun(SequentialFlow):
    Steps = [
        ...  # prior steps
        OpenROAD.DEFtoODB,
        ...  # subsequent steps
    ]
```
##### Configuration Example
```python
# Configure behavior in the flow's config dict:
config = {
    "PNR_CORNERS": None,  # (Optional[List[str]]) A list of fully-qualified IPVT corners to use during PnR. If unspecified, the...
    "SET_RC_VERBOSE": False,  # (bool) If set to true, set_rc commands are echoed. Quite noisy, but may be useful fo...
    "LAYERS_RC": None,  # (Optional[Dict[str, Dict[str, Dict[str, decimal.Decimal]]]]) Used during PNR steps, Specific custom resistance and capacitance values for ...
    "VIAS_R": None,  # (Optional[Dict[str, Dict[str, Dict[str, decimal.Decimal]]]]) Used during PNR steps, Specific custom resistance values for via layers. For ...
    "SIGNAL_WIRE_RC_LAYERS": None,  # (Optional[List[str]]) Sets estimated signal wire RC values to the average of these layers'. If you ...
    "CLOCK_WIRE_RC_LAYERS": None,  # (Optional[List[str]]) Sets estimated clock wire RC values to the average of these layers'. If you p...
    "PDN_CONNECT_MACROS_TO_GRID": True,  # (bool) Enables the connection of macros to the top level power grid.
    "PDN_MACRO_CONNECTIONS": None,  # (Optional[List[str]]) Specifies explicit power connections of internal macros to the top level powe...
    "PDN_ENABLE_GLOBAL_CONNECTIONS": True,  # (bool) Enables the creation of global connections in PDN generation.
    "PNR_SDC_FILE": None,  # (Optional[librelane.common.types.Path]) Specifies the SDC file used during all implementation (PnR) steps
    "STA_EXTRA_CORNER_TCL_FILE": None,  # (Optional[librelane.common.types.Path]) Experimental: specifies a additional configuration .tcl file to be called dur...
    "DEDUPLICATE_CORNERS": False,  # (bool) Cull duplicate IPVT corners during PNR, i.e. corners that share the same set ...
}
```

#### `OpenROAD.DetailedPlacement`
Performs "detailed placement" on an ODB file with global placement. This results
in a concrete and legal placement of all cells.

* **Inputs**: `odb` (OpenDB Database, `.odb`)
* **Outputs**: `odb` (OpenDB Database, `.odb`), `def` (Design Exchange Format, `.def`), `sdc` (Design Constraints, `.sdc`), `nl` (Verilog Netlist, `.nl.v`), `pnl` (Powered Verilog Netlist, `.pnl.v`)
* **Precursor Steps (Dependencies)**:
* Requires `odb` (OpenDB Database, `.odb`), produced by: `OpenROAD.Floorplan`, `Odb.ApplyDEFTemplate` (or other steps in Odb, OpenROAD)
* **Relevant Step-Specific Config**: `CLOCK_WIRE_RC_LAYERS`, `DEDUPLICATE_CORNERS`, `DPL_CELL_PADDING`, `LAYERS_RC`, `PDN_CONNECT_MACROS_TO_GRID`, `PDN_ENABLE_GLOBAL_CONNECTIONS`, `PDN_MACRO_CONNECTIONS`, `PL_MAX_DISPLACEMENT_X`, `PL_MAX_DISPLACEMENT_Y`, `PL_OPTIMIZE_MIRRORING`, `PNR_CORNERS`, `PNR_SDC_FILE`, `SET_RC_VERBOSE`, `SIGNAL_WIRE_RC_LAYERS`, `STA_EXTRA_CORNER_TCL_FILE`, `VIAS_R`

##### Flow Python Integration
```python
# Add this step to your flow's sequential Steps list:
class CustomRun(SequentialFlow):
    Steps = [
        ...  # prior steps
        OpenROAD.DetailedPlacement,
        ...  # subsequent steps
    ]
```
##### Configuration Example
```python
# Configure behavior in the flow's config dict:
config = {
    "PNR_CORNERS": None,  # (Optional[List[str]]) A list of fully-qualified IPVT corners to use during PnR. If unspecified, the...
    "SET_RC_VERBOSE": False,  # (bool) If set to true, set_rc commands are echoed. Quite noisy, but may be useful fo...
    "LAYERS_RC": None,  # (Optional[Dict[str, Dict[str, Dict[str, decimal.Decimal]]]]) Used during PNR steps, Specific custom resistance and capacitance values for ...
    "VIAS_R": None,  # (Optional[Dict[str, Dict[str, Dict[str, decimal.Decimal]]]]) Used during PNR steps, Specific custom resistance values for via layers. For ...
    "SIGNAL_WIRE_RC_LAYERS": None,  # (Optional[List[str]]) Sets estimated signal wire RC values to the average of these layers'. If you ...
    "CLOCK_WIRE_RC_LAYERS": None,  # (Optional[List[str]]) Sets estimated clock wire RC values to the average of these layers'. If you p...
    "PDN_CONNECT_MACROS_TO_GRID": True,  # (bool) Enables the connection of macros to the top level power grid.
    "PDN_MACRO_CONNECTIONS": None,  # (Optional[List[str]]) Specifies explicit power connections of internal macros to the top level powe...
    "PDN_ENABLE_GLOBAL_CONNECTIONS": True,  # (bool) Enables the creation of global connections in PDN generation.
    "PNR_SDC_FILE": None,  # (Optional[librelane.common.types.Path]) Specifies the SDC file used during all implementation (PnR) steps
    "STA_EXTRA_CORNER_TCL_FILE": None,  # (Optional[librelane.common.types.Path]) Experimental: specifies a additional configuration .tcl file to be called dur...
    "DEDUPLICATE_CORNERS": False,  # (bool) Cull duplicate IPVT corners during PNR, i.e. corners that share the same set ...
    "PL_OPTIMIZE_MIRRORING": True,  # (bool) Specifies whether or not to run an optimize_mirroring pass whenever detailed ...
    "PL_MAX_DISPLACEMENT_X": 500,  # (int) Specifies how far an instance can be moved along the X-axis when finding a si...
    "PL_MAX_DISPLACEMENT_Y": 100,  # (int) Specifies how far an instance can be moved along the Y-axis when finding a si...
    "DPL_CELL_PADDING": None,  # (int) Cell padding value (in sites) for detailed placement. The number will be inte...
}
```

#### `OpenROAD.DetailedRouting`
The latter phase of routing. This transforms the abstract nets from global
routing into wires on the metal layers that respect all design rules, avoids
creating accidental shorts, and ensures all wires are connected.

This is by far the longest part of a typical flow, taking hours, days or weeks
on larger designs.

After this point, all cells connected to a net can no longer be moved or
removed without a custom-written step of some kind that will also rip up
wires.

* **Inputs**: `odb` (OpenDB Database, `.odb`)
* **Outputs**: `odb` (OpenDB Database, `.odb`), `def` (Design Exchange Format, `.def`), `sdc` (Design Constraints, `.sdc`), `nl` (Verilog Netlist, `.nl.v`), `pnl` (Powered Verilog Netlist, `.pnl.v`)
* **Precursor Steps (Dependencies)**:
* Requires `odb` (OpenDB Database, `.odb`), produced by: `OpenROAD.Floorplan`, `Odb.ApplyDEFTemplate` (or other steps in Odb, OpenROAD)
* **Relevant Step-Specific Config**: `CLOCK_WIRE_RC_LAYERS`, `DEDUPLICATE_CORNERS`, `DIODE_PADDING`, `DRT_ANTENNA_REPAIR_DIODE_ONLY`, `DRT_ANTENNA_REPAIR_ITERS`, `DRT_ANTENNA_REPAIR_JUMPER_ONLY`, `DRT_ANTENNA_REPAIR_MARGIN`, `DRT_ASSIGN_NDR`, `DRT_OPT_ITERS`, `DRT_SAVE_DRC_REPORT_ITERS`, `DRT_SAVE_SNAPSHOTS`, `DRT_THREADS`, `GRT_ADJUSTMENT`, `GRT_ALLOW_CONGESTION`, `GRT_ANTENNA_REPAIR_DIODE_ONLY`, `GRT_ANTENNA_REPAIR_ITERS`, `GRT_ANTENNA_REPAIR_JUMPER_ONLY`, `GRT_ANTENNA_REPAIR_MARGIN`, `GRT_LAYER_ADJUSTMENTS`, `GRT_MACRO_EXTENSION`, `GRT_OVERFLOW_ITERS`, `LAYERS_RC`, `NON_DEFAULT_RULES`, `PDN_CONNECT_MACROS_TO_GRID`, `PDN_ENABLE_GLOBAL_CONNECTIONS`, `PDN_MACRO_CONNECTIONS`, `PNR_CORNERS`, `PNR_SDC_FILE`, `RT_CLOCK_MAX_LAYER`, `RT_CLOCK_MIN_LAYER`, `SET_RC_VERBOSE`, `SIGNAL_WIRE_RC_LAYERS`, `STA_EXTRA_CORNER_TCL_FILE`, `VIAS_R`

##### Flow Python Integration
```python
# Add this step to your flow's sequential Steps list:
class CustomRun(SequentialFlow):
    Steps = [
        ...  # prior steps
        OpenROAD.DetailedRouting,
        ...  # subsequent steps
    ]
```
##### Configuration Example
```python
# Configure behavior in the flow's config dict:
config = {
    "PNR_CORNERS": None,  # (Optional[List[str]]) A list of fully-qualified IPVT corners to use during PnR. If unspecified, the...
    "SET_RC_VERBOSE": False,  # (bool) If set to true, set_rc commands are echoed. Quite noisy, but may be useful fo...
    "LAYERS_RC": None,  # (Optional[Dict[str, Dict[str, Dict[str, decimal.Decimal]]]]) Used during PNR steps, Specific custom resistance and capacitance values for ...
    "VIAS_R": None,  # (Optional[Dict[str, Dict[str, Dict[str, decimal.Decimal]]]]) Used during PNR steps, Specific custom resistance values for via layers. For ...
    "SIGNAL_WIRE_RC_LAYERS": None,  # (Optional[List[str]]) Sets estimated signal wire RC values to the average of these layers'. If you ...
    "CLOCK_WIRE_RC_LAYERS": None,  # (Optional[List[str]]) Sets estimated clock wire RC values to the average of these layers'. If you p...
    "PDN_CONNECT_MACROS_TO_GRID": True,  # (bool) Enables the connection of macros to the top level power grid.
    "PDN_MACRO_CONNECTIONS": None,  # (Optional[List[str]]) Specifies explicit power connections of internal macros to the top level powe...
    "PDN_ENABLE_GLOBAL_CONNECTIONS": True,  # (bool) Enables the creation of global connections in PDN generation.
    "PNR_SDC_FILE": None,  # (Optional[librelane.common.types.Path]) Specifies the SDC file used during all implementation (PnR) steps
    "STA_EXTRA_CORNER_TCL_FILE": None,  # (Optional[librelane.common.types.Path]) Experimental: specifies a additional configuration .tcl file to be called dur...
    "DEDUPLICATE_CORNERS": False,  # (bool) Cull duplicate IPVT corners during PNR, i.e. corners that share the same set ...
    "RT_CLOCK_MIN_LAYER": None,  # (Optional[str]) The name of lowest layer to be used in routing the clock net.
    "RT_CLOCK_MAX_LAYER": None,  # (Optional[str]) The name of highest layer to be used in routing the clock net.
    "GRT_ADJUSTMENT": 0.3,  # (decimal.Decimal) Reduction in the routing capacity of the edges between the cells in the globa...
    "GRT_MACRO_EXTENSION": 0,  # (int) Sets the number of GCells added to the blockages boundaries from macros. A GC...
    "GRT_LAYER_ADJUSTMENTS": None,  # (List[decimal.Decimal]) Layer-specific reductions in the routing capacity of the edges between the ce...
    "DIODE_PADDING": None,  # (Optional[int]) Diode cell padding; increases the width of diode cells during placement checks..
    "GRT_ALLOW_CONGESTION": False,  # (bool) Allow congestion during global routing
    "GRT_ANTENNA_REPAIR_ITERS": 3,  # (int) The maximum number of iterations for global antenna repairs.
    "GRT_OVERFLOW_ITERS": 50,  # (int) The maximum number of iterations waiting for the overflow to reach the desire...
    "GRT_ANTENNA_REPAIR_MARGIN": 10,  # (int) The margin to over fix antenna violations.
    "GRT_ANTENNA_REPAIR_JUMPER_ONLY": False,  # (bool) Only use jumpers to fix antenna violations. Cannot be used in conjunction wit...
    "GRT_ANTENNA_REPAIR_DIODE_ONLY": False,  # (bool) Only use antenna diodes to fix antenna violations. Cannot be used in conjunct...
    "DRT_THREADS": None,  # (Optional[int]) Specifies the number of threads to be used in OpenROAD Detailed Routing. If u...
    "DRT_OPT_ITERS": 64,  # (int) Specifies the maximum number of optimization iterations during Detailed Routi...
    "DRT_SAVE_SNAPSHOTS": False,  # (bool) Experimental: saves an odb snapshot of the layout each routing iteration. Thi...
    "DRT_ANTENNA_REPAIR_ITERS": 3,  # (int) The maximum number of iterations to run antenna repair. Set to a positive int...
    "DRT_ANTENNA_REPAIR_MARGIN": 10,  # (int) The margin to over fix antenna violations.
    "DRT_ANTENNA_REPAIR_JUMPER_ONLY": False,  # (bool) Only use jumpers to fix antenna violations. Cannot be used in conjunction wit...
    "DRT_ANTENNA_REPAIR_DIODE_ONLY": False,  # (bool) Only use antenna diodes to fix antenna violations. Cannot be used in conjunct...
    "DRT_SAVE_DRC_REPORT_ITERS": None,  # (Optional[int]) Write a DRC report every N iterations. If DRT_SAVE_SNAPSHOTS is enabled, ther...
    "NON_DEFAULT_RULES": None,  # (Optional[dict[str, librelane.steps.openroad.NDR]]) Specify non-default rules. Can be used to change the width, spacing and vias ...
    "DRT_ASSIGN_NDR": None,  # (Optional[dict[str, str]]) Specify which nets should be assigned to which non-default rule. The net name...
}
```

#### `OpenROAD.DumpRCValues`
Creates three reports:

* Initial Database Layer RC Values (from Tech LEF)
* Modified Database Layer RC Values
* Modified Resizer Layer RC Values

* **Inputs**: `def` (Design Exchange Format, `.def`)
* **Outputs**: `odb` (OpenDB Database, `.odb`), `def` (Design Exchange Format, `.def`), `sdc` (Design Constraints, `.sdc`), `nl` (Verilog Netlist, `.nl.v`), `pnl` (Powered Verilog Netlist, `.pnl.v`)
* **Precursor Steps (Dependencies)**:
* Requires `def` (Design Exchange Format, `.def`), produced by: `OpenROAD.Floorplan`, `Odb.ApplyDEFTemplate` (or other steps in Odb, OpenROAD)
* **Relevant Step-Specific Config**: `CLOCK_WIRE_RC_LAYERS`, `DEDUPLICATE_CORNERS`, `LAYERS_RC`, `PDN_CONNECT_MACROS_TO_GRID`, `PDN_ENABLE_GLOBAL_CONNECTIONS`, `PDN_MACRO_CONNECTIONS`, `PNR_CORNERS`, `PNR_SDC_FILE`, `SET_RC_VERBOSE`, `SIGNAL_WIRE_RC_LAYERS`, `STA_EXTRA_CORNER_TCL_FILE`, `VIAS_R`

##### Flow Python Integration
```python
# Add this step to your flow's sequential Steps list:
class CustomRun(SequentialFlow):
    Steps = [
        ...  # prior steps
        OpenROAD.DumpRCValues,
        ...  # subsequent steps
    ]
```
##### Configuration Example
```python
# Configure behavior in the flow's config dict:
config = {
    "PNR_CORNERS": None,  # (Optional[List[str]]) A list of fully-qualified IPVT corners to use during PnR. If unspecified, the...
    "SET_RC_VERBOSE": False,  # (bool) If set to true, set_rc commands are echoed. Quite noisy, but may be useful fo...
    "LAYERS_RC": None,  # (Optional[Dict[str, Dict[str, Dict[str, decimal.Decimal]]]]) Used during PNR steps, Specific custom resistance and capacitance values for ...
    "VIAS_R": None,  # (Optional[Dict[str, Dict[str, Dict[str, decimal.Decimal]]]]) Used during PNR steps, Specific custom resistance values for via layers. For ...
    "SIGNAL_WIRE_RC_LAYERS": None,  # (Optional[List[str]]) Sets estimated signal wire RC values to the average of these layers'. If you ...
    "CLOCK_WIRE_RC_LAYERS": None,  # (Optional[List[str]]) Sets estimated clock wire RC values to the average of these layers'. If you p...
    "PDN_CONNECT_MACROS_TO_GRID": True,  # (bool) Enables the connection of macros to the top level power grid.
    "PDN_MACRO_CONNECTIONS": None,  # (Optional[List[str]]) Specifies explicit power connections of internal macros to the top level powe...
    "PDN_ENABLE_GLOBAL_CONNECTIONS": True,  # (bool) Enables the creation of global connections in PDN generation.
    "PNR_SDC_FILE": None,  # (Optional[librelane.common.types.Path]) Specifies the SDC file used during all implementation (PnR) steps
    "STA_EXTRA_CORNER_TCL_FILE": None,  # (Optional[librelane.common.types.Path]) Experimental: specifies a additional configuration .tcl file to be called dur...
    "DEDUPLICATE_CORNERS": False,  # (bool) Cull duplicate IPVT corners during PNR, i.e. corners that share the same set ...
}
```

#### `OpenROAD.FillInsertion`
Fills gaps in the floorplan with filler and decap cells.

This is run after detailed placement. After this point, the design is basically
completely hardened.

* **Inputs**: `odb` (OpenDB Database, `.odb`)
* **Outputs**: `odb` (OpenDB Database, `.odb`), `def` (Design Exchange Format, `.def`), `sdc` (Design Constraints, `.sdc`), `nl` (Verilog Netlist, `.nl.v`), `pnl` (Powered Verilog Netlist, `.pnl.v`)
* **Precursor Steps (Dependencies)**:
* Requires `odb` (OpenDB Database, `.odb`), produced by: `OpenROAD.Floorplan`, `Odb.ApplyDEFTemplate` (or other steps in Odb, OpenROAD)
* **Relevant Step-Specific Config**: `CLOCK_WIRE_RC_LAYERS`, `DEDUPLICATE_CORNERS`, `LAYERS_RC`, `PDN_CONNECT_MACROS_TO_GRID`, `PDN_ENABLE_GLOBAL_CONNECTIONS`, `PDN_MACRO_CONNECTIONS`, `PNR_CORNERS`, `PNR_SDC_FILE`, `SET_RC_VERBOSE`, `SIGNAL_WIRE_RC_LAYERS`, `STA_EXTRA_CORNER_TCL_FILE`, `VIAS_R`

##### Flow Python Integration
```python
# Add this step to your flow's sequential Steps list:
class CustomRun(SequentialFlow):
    Steps = [
        ...  # prior steps
        OpenROAD.FillInsertion,
        ...  # subsequent steps
    ]
```
##### Configuration Example
```python
# Configure behavior in the flow's config dict:
config = {
    "PNR_CORNERS": None,  # (Optional[List[str]]) A list of fully-qualified IPVT corners to use during PnR. If unspecified, the...
    "SET_RC_VERBOSE": False,  # (bool) If set to true, set_rc commands are echoed. Quite noisy, but may be useful fo...
    "LAYERS_RC": None,  # (Optional[Dict[str, Dict[str, Dict[str, decimal.Decimal]]]]) Used during PNR steps, Specific custom resistance and capacitance values for ...
    "VIAS_R": None,  # (Optional[Dict[str, Dict[str, Dict[str, decimal.Decimal]]]]) Used during PNR steps, Specific custom resistance values for via layers. For ...
    "SIGNAL_WIRE_RC_LAYERS": None,  # (Optional[List[str]]) Sets estimated signal wire RC values to the average of these layers'. If you ...
    "CLOCK_WIRE_RC_LAYERS": None,  # (Optional[List[str]]) Sets estimated clock wire RC values to the average of these layers'. If you p...
    "PDN_CONNECT_MACROS_TO_GRID": True,  # (bool) Enables the connection of macros to the top level power grid.
    "PDN_MACRO_CONNECTIONS": None,  # (Optional[List[str]]) Specifies explicit power connections of internal macros to the top level powe...
    "PDN_ENABLE_GLOBAL_CONNECTIONS": True,  # (bool) Enables the creation of global connections in PDN generation.
    "PNR_SDC_FILE": None,  # (Optional[librelane.common.types.Path]) Specifies the SDC file used during all implementation (PnR) steps
    "STA_EXTRA_CORNER_TCL_FILE": None,  # (Optional[librelane.common.types.Path]) Experimental: specifies a additional configuration .tcl file to be called dur...
    "DEDUPLICATE_CORNERS": False,  # (bool) Cull duplicate IPVT corners during PNR, i.e. corners that share the same set ...
}
```

#### `OpenROAD.Floorplan`
Creates DEF and ODB files with the initial floorplan based on the Yosys netlist.

* **Inputs**: `nl` (Verilog Netlist, `.nl.v`)
* **Outputs**: `odb` (OpenDB Database, `.odb`), `def` (Design Exchange Format, `.def`), `sdc` (Design Constraints, `.sdc`), `nl` (Verilog Netlist, `.nl.v`), `pnl` (Powered Verilog Netlist, `.pnl.v`)
* **Precursor Steps (Dependencies)**:
* Requires `nl` (Verilog Netlist, `.nl.v`), produced by: `Yosys.Synthesis`, `OpenROAD.Floorplan`, `Odb.DiodesOnPorts` (or other steps in Odb, OpenROAD, Yosys)
* **Relevant Step-Specific Config**: `BOTTOM_MARGIN_MULT`, `CLOCK_WIRE_RC_LAYERS`, `CORE_AREA`, `DEDUPLICATE_CORNERS`, `EXTRA_SITES`, `FP_ASPECT_RATIO`, `FP_CORE_UTIL`, `FP_FLIP_SITES`, `FP_OBSTRUCTIONS`, `FP_SIZING`, `FP_TRACKS_INFO`, `LAYERS_RC`, `LEFT_MARGIN_MULT`, `PDN_CONNECT_MACROS_TO_GRID`, `PDN_ENABLE_GLOBAL_CONNECTIONS`, `PDN_MACRO_CONNECTIONS`, `PL_SOFT_OBSTRUCTIONS`, `PNR_CORNERS`, `PNR_SDC_FILE`, `RIGHT_MARGIN_MULT`, `SET_RC_VERBOSE`, `SIGNAL_WIRE_RC_LAYERS`, `STA_EXTRA_CORNER_TCL_FILE`, `TOP_MARGIN_MULT`, `VIAS_R`

##### Flow Python Integration
```python
# Add this step to your flow's sequential Steps list:
class CustomRun(SequentialFlow):
    Steps = [
        ...  # prior steps
        OpenROAD.Floorplan,
        ...  # subsequent steps
    ]
```
##### Configuration Example
```python
# Configure behavior in the flow's config dict:
config = {
    "PNR_CORNERS": None,  # (Optional[List[str]]) A list of fully-qualified IPVT corners to use during PnR. If unspecified, the...
    "SET_RC_VERBOSE": False,  # (bool) If set to true, set_rc commands are echoed. Quite noisy, but may be useful fo...
    "LAYERS_RC": None,  # (Optional[Dict[str, Dict[str, Dict[str, decimal.Decimal]]]]) Used during PNR steps, Specific custom resistance and capacitance values for ...
    "VIAS_R": None,  # (Optional[Dict[str, Dict[str, Dict[str, decimal.Decimal]]]]) Used during PNR steps, Specific custom resistance values for via layers. For ...
    "SIGNAL_WIRE_RC_LAYERS": None,  # (Optional[List[str]]) Sets estimated signal wire RC values to the average of these layers'. If you ...
    "CLOCK_WIRE_RC_LAYERS": None,  # (Optional[List[str]]) Sets estimated clock wire RC values to the average of these layers'. If you p...
    "PDN_CONNECT_MACROS_TO_GRID": True,  # (bool) Enables the connection of macros to the top level power grid.
    "PDN_MACRO_CONNECTIONS": None,  # (Optional[List[str]]) Specifies explicit power connections of internal macros to the top level powe...
    "PDN_ENABLE_GLOBAL_CONNECTIONS": True,  # (bool) Enables the creation of global connections in PDN generation.
    "PNR_SDC_FILE": None,  # (Optional[librelane.common.types.Path]) Specifies the SDC file used during all implementation (PnR) steps
    "STA_EXTRA_CORNER_TCL_FILE": None,  # (Optional[librelane.common.types.Path]) Experimental: specifies a additional configuration .tcl file to be called dur...
    "DEDUPLICATE_CORNERS": False,  # (bool) Cull duplicate IPVT corners during PNR, i.e. corners that share the same set ...
    "FP_FLIP_SITES": None,  # (Optional[List[str]]) Flip these sites vertically. Useful in niche alignment scenarios where single...
    "FP_TRACKS_INFO": None,  # (librelane.common.types.Path) A path to the a classic OpenROAD `.tracks` file. Used by the floorplanner to ...
    "FP_SIZING": 'relative',  # (Literal['absolute', 'relative']) Sizing mode for floorplanning
    "FP_ASPECT_RATIO": 1,  # (decimal.Decimal) The core's aspect ratio (height / width).
    "FP_CORE_UTIL": 50,  # (decimal.Decimal) The core utilization percentage.
    "FP_OBSTRUCTIONS": None,  # (Optional[List[Tuple[decimal.Decimal, decimal.Decimal, decimal.Decimal, decimal.Decimal]]]) Obstructions applied at floorplanning stage. Placement sites are never genera...
    "PL_SOFT_OBSTRUCTIONS": None,  # (Optional[List[Tuple[decimal.Decimal, decimal.Decimal, decimal.Decimal, decimal.Decimal]]]) Soft placement blockages applied at the floorplanning stage. Areas that are s...
    "CORE_AREA": None,  # (Optional[Tuple[decimal.Decimal, decimal.Decimal, decimal.Decimal, decimal.Decimal]]) Specifies a core area (i.e. die area minus margins) to be used in floorplanni...
    "BOTTOM_MARGIN_MULT": 4,  # (decimal.Decimal) The core margin, in multiples of site heights, from the bottom boundary. If `...
    "TOP_MARGIN_MULT": 4,  # (decimal.Decimal) The core margin, in multiples of site heights, from the top boundary. If `DIE...
    "LEFT_MARGIN_MULT": 12,  # (decimal.Decimal) The core margin, in multiples of site widths, from the left boundary. If `DIE...
    "RIGHT_MARGIN_MULT": 12,  # (decimal.Decimal) The core margin, in multiples of site widths, from the right boundary. If `DI...
    "EXTRA_SITES": None,  # (Optional[List[str]]) Explicitly specify sites other than `PLACE_SITE` to create rows for. If the a...
}
```

#### `OpenROAD.GeneratePDN`
Creates a power distribution network on a floorplanned ODB file.

* **Inputs**: `odb` (OpenDB Database, `.odb`)
* **Outputs**: `odb` (OpenDB Database, `.odb`), `def` (Design Exchange Format, `.def`), `sdc` (Design Constraints, `.sdc`), `nl` (Verilog Netlist, `.nl.v`), `pnl` (Powered Verilog Netlist, `.pnl.v`)
* **Precursor Steps (Dependencies)**:
* Requires `odb` (OpenDB Database, `.odb`), produced by: `OpenROAD.Floorplan`, `Odb.ApplyDEFTemplate` (or other steps in Odb, OpenROAD)
* **Relevant Step-Specific Config**: `CLOCK_WIRE_RC_LAYERS`, `DEDUPLICATE_CORNERS`, `LAYERS_RC`, `PDN_CFG`, `PDN_CONNECT_MACROS_TO_GRID`, `PDN_CORE_HORIZONTAL_LAYER`, `PDN_CORE_RING`, `PDN_CORE_RING_ALLOW_OUT_OF_DIE`, `PDN_CORE_RING_CONNECT_TO_PADS`, `PDN_CORE_RING_HOFFSET`, `PDN_CORE_RING_HSPACING`, `PDN_CORE_RING_HWIDTH`, `PDN_CORE_RING_VOFFSET`, `PDN_CORE_RING_VSPACING`, `PDN_CORE_RING_VWIDTH`, `PDN_CORE_VERTICAL_LAYER`, `PDN_ENABLE_GLOBAL_CONNECTIONS`, `PDN_ENABLE_PINS`, `PDN_ENABLE_RAILS`, `PDN_EXTEND_TO`, `PDN_HOFFSET`, `PDN_HORIZONTAL_HALO`, `PDN_HORIZONTAL_LAYER`, `PDN_HPITCH`, `PDN_HSPACING`, `PDN_HWIDTH`, `PDN_MACRO_CONNECTIONS`, `PDN_MULTILAYER`, `PDN_RAIL_LAYER`, `PDN_RAIL_OFFSET`, `PDN_RAIL_WIDTH`, `PDN_SKIPTRIM`, `PDN_VERTICAL_HALO`, `PDN_VERTICAL_LAYER`, `PDN_VOFFSET`, `PDN_VPITCH`, `PDN_VSPACING`, `PDN_VWIDTH`, `PNR_CORNERS`, `PNR_SDC_FILE`, `SET_RC_VERBOSE`, `SIGNAL_WIRE_RC_LAYERS`, `STA_EXTRA_CORNER_TCL_FILE`, `VIAS_R`

##### Flow Python Integration
```python
# Add this step to your flow's sequential Steps list:
class CustomRun(SequentialFlow):
    Steps = [
        ...  # prior steps
        OpenROAD.GeneratePDN,
        ...  # subsequent steps
    ]
```
##### Configuration Example
```python
# Configure behavior in the flow's config dict:
config = {
    "PNR_CORNERS": None,  # (Optional[List[str]]) A list of fully-qualified IPVT corners to use during PnR. If unspecified, the...
    "SET_RC_VERBOSE": False,  # (bool) If set to true, set_rc commands are echoed. Quite noisy, but may be useful fo...
    "LAYERS_RC": None,  # (Optional[Dict[str, Dict[str, Dict[str, decimal.Decimal]]]]) Used during PNR steps, Specific custom resistance and capacitance values for ...
    "VIAS_R": None,  # (Optional[Dict[str, Dict[str, Dict[str, decimal.Decimal]]]]) Used during PNR steps, Specific custom resistance values for via layers. For ...
    "SIGNAL_WIRE_RC_LAYERS": None,  # (Optional[List[str]]) Sets estimated signal wire RC values to the average of these layers'. If you ...
    "CLOCK_WIRE_RC_LAYERS": None,  # (Optional[List[str]]) Sets estimated clock wire RC values to the average of these layers'. If you p...
    "PDN_CONNECT_MACROS_TO_GRID": True,  # (bool) Enables the connection of macros to the top level power grid.
    "PDN_MACRO_CONNECTIONS": None,  # (Optional[List[str]]) Specifies explicit power connections of internal macros to the top level powe...
    "PDN_ENABLE_GLOBAL_CONNECTIONS": True,  # (bool) Enables the creation of global connections in PDN generation.
    "PNR_SDC_FILE": None,  # (Optional[librelane.common.types.Path]) Specifies the SDC file used during all implementation (PnR) steps
    "STA_EXTRA_CORNER_TCL_FILE": None,  # (Optional[librelane.common.types.Path]) Experimental: specifies a additional configuration .tcl file to be called dur...
    "DEDUPLICATE_CORNERS": False,  # (bool) Cull duplicate IPVT corners during PNR, i.e. corners that share the same set ...
    "PDN_SKIPTRIM": False,  # (bool) Enables `-skip_trim` option during pdngen which skips the metal trim step, wh...
    "PDN_CORE_RING": False,  # (bool) Enables adding a core ring around the design. More details on the control var...
    "PDN_ENABLE_RAILS": True,  # (bool) Enables the creation of rails in the power grid.
    "PDN_HORIZONTAL_HALO": 10,  # (decimal.Decimal) Sets the horizontal halo around the macros during power grid insertion.
    "PDN_VERTICAL_HALO": 10,  # (decimal.Decimal) Sets the vertical halo around the macros during power grid insertion.
    "PDN_MULTILAYER": True,  # (bool) Controls the layers used in the power grid. If set to false, only the lower l...
    "PDN_RAIL_OFFSET": None,  # (decimal.Decimal) The offset for the power distribution network rails for first metal layer.
    "PDN_VWIDTH": None,  # (decimal.Decimal) The strap width for the vertical layer in generated power distribution networks.
    "PDN_HWIDTH": None,  # (decimal.Decimal) The strap width for the horizontal layer in generated power distribution netw...
    "PDN_VSPACING": None,  # (decimal.Decimal) Intra-spacing (within a set) of vertical straps in generated power distributi...
    "PDN_HSPACING": None,  # (decimal.Decimal) Intra-spacing (within a set) of horizontal straps in generated power distribu...
    "PDN_VPITCH": None,  # (decimal.Decimal) Inter-distance (between sets) of vertical power straps in generated power dis...
    "PDN_HPITCH": None,  # (decimal.Decimal) Inter-distance (between sets) of horizontal power straps in generated power d...
    "PDN_VOFFSET": None,  # (decimal.Decimal) Initial offset for sets of vertical power straps.
    "PDN_HOFFSET": None,  # (decimal.Decimal) Initial offset for sets of horizontal power straps.
    "PDN_CORE_RING_VWIDTH": None,  # (decimal.Decimal) The width for the vertical layer in the core ring of generated power distribu...
    "PDN_CORE_RING_HWIDTH": None,  # (decimal.Decimal) The width for the horizontal layer in the core ring of generated power distri...
    "PDN_CORE_RING_VSPACING": None,  # (decimal.Decimal) The spacing for the vertical layer in the core ring of generated power distri...
    "PDN_CORE_RING_HSPACING": None,  # (decimal.Decimal) The spacing for the horizontal layer in the core ring of generated power dist...
    "PDN_CORE_RING_VOFFSET": None,  # (decimal.Decimal) The offset for the vertical layer in the core ring of generated power distrib...
    "PDN_CORE_RING_HOFFSET": None,  # (decimal.Decimal) The offset for the horizontal layer in the core ring of generated power distr...
    "PDN_CORE_RING_CONNECT_TO_PADS": False,  # (bool) If specified, the core side of the pad pins will be connected to the ring.
    "PDN_CORE_RING_ALLOW_OUT_OF_DIE": True,  # (bool) If specified, the ring shapes are allowed to be outside the die boundary.
    "PDN_RAIL_LAYER": None,  # (str) Defines the metal layer used for PDN rails.
    "PDN_RAIL_WIDTH": None,  # (decimal.Decimal) Defines the width of PDN rails on the `PDN_RAIL_LAYER` layer.
    "PDN_HORIZONTAL_LAYER": None,  # (str) Defines the horizontal PDN layer.
    "PDN_VERTICAL_LAYER": None,  # (str) Defines the vertical PDN layer.
    "PDN_CORE_HORIZONTAL_LAYER": None,  # (Optional[str]) Defines the horizontal PDN layer for the core ring. Falls back to `PDN_HORIZO...
    "PDN_CORE_VERTICAL_LAYER": None,  # (Optional[str]) Defines the vertical PDN layer for the core ring. Falls back to `PDN_VERTICAL...
    "PDN_EXTEND_TO": 'core_ring',  # (Literal['core_ring', 'boundary']) Defines how far the stripes and rings extend.
    "PDN_ENABLE_PINS": True,  # (bool) If specified, the power straps will be promoted to block pins.
    "PDN_CFG": None,  # (Optional[librelane.common.types.Path]) A custom PDN configuration file. If not provided, the default PDN config will...
}
```

#### `OpenROAD.GlobalPlacement`
Performs a somewhat nebulous initial placement for standard cells in a
floorplan. While the placement is not concrete, it is enough to start
accounting for issues such as fanout, transition time, et cetera.

* **Inputs**: `odb` (OpenDB Database, `.odb`)
* **Outputs**: `odb` (OpenDB Database, `.odb`), `def` (Design Exchange Format, `.def`), `sdc` (Design Constraints, `.sdc`), `nl` (Verilog Netlist, `.nl.v`), `pnl` (Powered Verilog Netlist, `.pnl.v`)
* **Precursor Steps (Dependencies)**:
* Requires `odb` (OpenDB Database, `.odb`), produced by: `OpenROAD.Floorplan`, `Odb.ApplyDEFTemplate` (or other steps in Odb, OpenROAD)
* **Relevant Step-Specific Config**: `CLOCK_WIRE_RC_LAYERS`, `DEDUPLICATE_CORNERS`, `DPL_CELL_PADDING`, `FP_CORE_UTIL`, `GPL_CELL_PADDING`, `GRT_ADJUSTMENT`, `GRT_LAYER_ADJUSTMENTS`, `GRT_MACRO_EXTENSION`, `LAYERS_RC`, `PDN_CONNECT_MACROS_TO_GRID`, `PDN_ENABLE_GLOBAL_CONNECTIONS`, `PDN_MACRO_CONNECTIONS`, `PL_KEEP_RESIZE_BELOW_OVERFLOW`, `PL_MAX_DISPLACEMENT_X`, `PL_MAX_DISPLACEMENT_Y`, `PL_MAX_PHI_COEFFICIENT`, `PL_MIN_PHI_COEFFICIENT`, `PL_OPTIMIZE_MIRRORING`, `PL_ROUTABILITY_DRIVEN`, `PL_ROUTABILITY_OVERFLOW_THRESHOLD`, `PL_SKIP_INITIAL_PLACEMENT`, `PL_TARGET_DENSITY_PCT`, `PL_TIMING_DRIVEN`, `PL_WIRE_LENGTH_COEF`, `PNR_CORNERS`, `PNR_SDC_FILE`, `RSZ_CORNERS`, `RSZ_DONT_TOUCH_LIST`, `RSZ_DONT_TOUCH_RX`, `RT_CLOCK_MAX_LAYER`, `RT_CLOCK_MIN_LAYER`, `SET_RC_VERBOSE`, `SIGNAL_WIRE_RC_LAYERS`, `STA_EXTRA_CORNER_TCL_FILE`, `VIAS_R`

##### Flow Python Integration
```python
# Add this step to your flow's sequential Steps list:
class CustomRun(SequentialFlow):
    Steps = [
        ...  # prior steps
        OpenROAD.GlobalPlacement,
        ...  # subsequent steps
    ]
```
##### Configuration Example
```python
# Configure behavior in the flow's config dict:
config = {
    "PNR_CORNERS": None,  # (Optional[List[str]]) A list of fully-qualified IPVT corners to use during PnR. If unspecified, the...
    "SET_RC_VERBOSE": False,  # (bool) If set to true, set_rc commands are echoed. Quite noisy, but may be useful fo...
    "LAYERS_RC": None,  # (Optional[Dict[str, Dict[str, Dict[str, decimal.Decimal]]]]) Used during PNR steps, Specific custom resistance and capacitance values for ...
    "VIAS_R": None,  # (Optional[Dict[str, Dict[str, Dict[str, decimal.Decimal]]]]) Used during PNR steps, Specific custom resistance values for via layers. For ...
    "SIGNAL_WIRE_RC_LAYERS": None,  # (Optional[List[str]]) Sets estimated signal wire RC values to the average of these layers'. If you ...
    "CLOCK_WIRE_RC_LAYERS": None,  # (Optional[List[str]]) Sets estimated clock wire RC values to the average of these layers'. If you p...
    "PDN_CONNECT_MACROS_TO_GRID": True,  # (bool) Enables the connection of macros to the top level power grid.
    "PDN_MACRO_CONNECTIONS": None,  # (Optional[List[str]]) Specifies explicit power connections of internal macros to the top level powe...
    "PDN_ENABLE_GLOBAL_CONNECTIONS": True,  # (bool) Enables the creation of global connections in PDN generation.
    "PNR_SDC_FILE": None,  # (Optional[librelane.common.types.Path]) Specifies the SDC file used during all implementation (PnR) steps
    "STA_EXTRA_CORNER_TCL_FILE": None,  # (Optional[librelane.common.types.Path]) Experimental: specifies a additional configuration .tcl file to be called dur...
    "DEDUPLICATE_CORNERS": False,  # (bool) Cull duplicate IPVT corners during PNR, i.e. corners that share the same set ...
    "RT_CLOCK_MIN_LAYER": None,  # (Optional[str]) The name of lowest layer to be used in routing the clock net.
    "RT_CLOCK_MAX_LAYER": None,  # (Optional[str]) The name of highest layer to be used in routing the clock net.
    "GRT_ADJUSTMENT": 0.3,  # (decimal.Decimal) Reduction in the routing capacity of the edges between the cells in the globa...
    "GRT_MACRO_EXTENSION": 0,  # (int) Sets the number of GCells added to the blockages boundaries from macros. A GC...
    "GRT_LAYER_ADJUSTMENTS": None,  # (List[decimal.Decimal]) Layer-specific reductions in the routing capacity of the edges between the ce...
    "PL_OPTIMIZE_MIRRORING": True,  # (bool) Specifies whether or not to run an optimize_mirroring pass whenever detailed ...
    "PL_MAX_DISPLACEMENT_X": 500,  # (int) Specifies how far an instance can be moved along the X-axis when finding a si...
    "PL_MAX_DISPLACEMENT_Y": 100,  # (int) Specifies how far an instance can be moved along the Y-axis when finding a si...
    "DPL_CELL_PADDING": None,  # (int) Cell padding value (in sites) for detailed placement. The number will be inte...
    "RSZ_DONT_TOUCH_RX": '\$^',  # (str) A single regular expression designating nets or instances as "don't touch" by...
    "RSZ_DONT_TOUCH_LIST": None,  # (Optional[List[str]]) A list of nets and instances as "don't touch" by design repairs or resizer op...
    "RSZ_CORNERS": None,  # (Optional[List[str]]) Resizer step-specific override for PNR_CORNERS.
    "PL_TARGET_DENSITY_PCT": None,  # (Optional[decimal.Decimal]) The desired placement density of cells. If not specified, the value will be e...
    "PL_SKIP_INITIAL_PLACEMENT": False,  # (bool) Specifies whether the placer should run initial placement or not.
    "PL_WIRE_LENGTH_COEF": 0.25,  # (decimal.Decimal) Global placement initial wirelength coefficient. Decreasing the variable will...
    "PL_MIN_PHI_COEFFICIENT": None,  # (Optional[decimal.Decimal]) Sets a lower bound on the µ_k variable in the GPL algorithm. Useful if global...
    "PL_MAX_PHI_COEFFICIENT": None,  # (Optional[decimal.Decimal]) Sets a upper bound on the µ_k variable in the GPL algorithm. Useful if global...
    "FP_CORE_UTIL": 50,  # (decimal.Decimal) The core utilization percentage.
    "GPL_CELL_PADDING": None,  # (int) Cell padding value (in sites) for global placement. The number will be intege...
    "PL_KEEP_RESIZE_BELOW_OVERFLOW": None,  # (Optional[decimal.Decimal]) Only applicable when PL_TIMING_DRIVEN is enabled. When the overflow is below ...
    "PL_TIMING_DRIVEN": False,  # (bool) Specifies whether the placer should use timing-driven placement.
    "PL_ROUTABILITY_DRIVEN": True,  # (bool) Specifies whether the placer should use routability driven placement.
    "PL_ROUTABILITY_OVERFLOW_THRESHOLD": None,  # (Optional[decimal.Decimal]) Sets overflow threshold for routability mode.
}
```

#### `OpenROAD.GlobalPlacementSkipIO`
Performs preliminary global placement as a basis for pin placement.

This is useful for flows where the:
* Cells are placed
* I/Os are placed to match the cells
* Cells are then re-placed for an optimal placement

* **Inputs**: `odb` (OpenDB Database, `.odb`)
* **Outputs**: `odb` (OpenDB Database, `.odb`), `def` (Design Exchange Format, `.def`), `sdc` (Design Constraints, `.sdc`), `nl` (Verilog Netlist, `.nl.v`), `pnl` (Powered Verilog Netlist, `.pnl.v`)
* **Precursor Steps (Dependencies)**:
* Requires `odb` (OpenDB Database, `.odb`), produced by: `OpenROAD.Floorplan`, `Odb.ApplyDEFTemplate` (or other steps in Odb, OpenROAD)
* **Relevant Step-Specific Config**: `CLOCK_WIRE_RC_LAYERS`, `DEDUPLICATE_CORNERS`, `DPL_CELL_PADDING`, `FP_CORE_UTIL`, `FP_DEF_TEMPLATE`, `GPL_CELL_PADDING`, `GRT_ADJUSTMENT`, `GRT_LAYER_ADJUSTMENTS`, `GRT_MACRO_EXTENSION`, `IO_PIN_ORDER_CFG`, `IO_PIN_PLACEMENT_MODE`, `LAYERS_RC`, `PDN_CONNECT_MACROS_TO_GRID`, `PDN_ENABLE_GLOBAL_CONNECTIONS`, `PDN_MACRO_CONNECTIONS`, `PL_KEEP_RESIZE_BELOW_OVERFLOW`, `PL_MAX_DISPLACEMENT_X`, `PL_MAX_DISPLACEMENT_Y`, `PL_MAX_PHI_COEFFICIENT`, `PL_MIN_PHI_COEFFICIENT`, `PL_OPTIMIZE_MIRRORING`, `PL_SKIP_INITIAL_PLACEMENT`, `PL_TARGET_DENSITY_PCT`, `PL_WIRE_LENGTH_COEF`, `PNR_CORNERS`, `PNR_SDC_FILE`, `RSZ_CORNERS`, `RSZ_DONT_TOUCH_LIST`, `RSZ_DONT_TOUCH_RX`, `RT_CLOCK_MAX_LAYER`, `RT_CLOCK_MIN_LAYER`, `SET_RC_VERBOSE`, `SIGNAL_WIRE_RC_LAYERS`, `STA_EXTRA_CORNER_TCL_FILE`, `VIAS_R`

##### Flow Python Integration
```python
# Add this step to your flow's sequential Steps list:
class CustomRun(SequentialFlow):
    Steps = [
        ...  # prior steps
        OpenROAD.GlobalPlacementSkipIO,
        ...  # subsequent steps
    ]
```
##### Configuration Example
```python
# Configure behavior in the flow's config dict:
config = {
    "PNR_CORNERS": None,  # (Optional[List[str]]) A list of fully-qualified IPVT corners to use during PnR. If unspecified, the...
    "SET_RC_VERBOSE": False,  # (bool) If set to true, set_rc commands are echoed. Quite noisy, but may be useful fo...
    "LAYERS_RC": None,  # (Optional[Dict[str, Dict[str, Dict[str, decimal.Decimal]]]]) Used during PNR steps, Specific custom resistance and capacitance values for ...
    "VIAS_R": None,  # (Optional[Dict[str, Dict[str, Dict[str, decimal.Decimal]]]]) Used during PNR steps, Specific custom resistance values for via layers. For ...
    "SIGNAL_WIRE_RC_LAYERS": None,  # (Optional[List[str]]) Sets estimated signal wire RC values to the average of these layers'. If you ...
    "CLOCK_WIRE_RC_LAYERS": None,  # (Optional[List[str]]) Sets estimated clock wire RC values to the average of these layers'. If you p...
    "PDN_CONNECT_MACROS_TO_GRID": True,  # (bool) Enables the connection of macros to the top level power grid.
    "PDN_MACRO_CONNECTIONS": None,  # (Optional[List[str]]) Specifies explicit power connections of internal macros to the top level powe...
    "PDN_ENABLE_GLOBAL_CONNECTIONS": True,  # (bool) Enables the creation of global connections in PDN generation.
    "PNR_SDC_FILE": None,  # (Optional[librelane.common.types.Path]) Specifies the SDC file used during all implementation (PnR) steps
    "STA_EXTRA_CORNER_TCL_FILE": None,  # (Optional[librelane.common.types.Path]) Experimental: specifies a additional configuration .tcl file to be called dur...
    "DEDUPLICATE_CORNERS": False,  # (bool) Cull duplicate IPVT corners during PNR, i.e. corners that share the same set ...
    "RT_CLOCK_MIN_LAYER": None,  # (Optional[str]) The name of lowest layer to be used in routing the clock net.
    "RT_CLOCK_MAX_LAYER": None,  # (Optional[str]) The name of highest layer to be used in routing the clock net.
    "GRT_ADJUSTMENT": 0.3,  # (decimal.Decimal) Reduction in the routing capacity of the edges between the cells in the globa...
    "GRT_MACRO_EXTENSION": 0,  # (int) Sets the number of GCells added to the blockages boundaries from macros. A GC...
    "GRT_LAYER_ADJUSTMENTS": None,  # (List[decimal.Decimal]) Layer-specific reductions in the routing capacity of the edges between the ce...
    "PL_OPTIMIZE_MIRRORING": True,  # (bool) Specifies whether or not to run an optimize_mirroring pass whenever detailed ...
    "PL_MAX_DISPLACEMENT_X": 500,  # (int) Specifies how far an instance can be moved along the X-axis when finding a si...
    "PL_MAX_DISPLACEMENT_Y": 100,  # (int) Specifies how far an instance can be moved along the Y-axis when finding a si...
    "DPL_CELL_PADDING": None,  # (int) Cell padding value (in sites) for detailed placement. The number will be inte...
    "RSZ_DONT_TOUCH_RX": '\$^',  # (str) A single regular expression designating nets or instances as "don't touch" by...
    "RSZ_DONT_TOUCH_LIST": None,  # (Optional[List[str]]) A list of nets and instances as "don't touch" by design repairs or resizer op...
    "RSZ_CORNERS": None,  # (Optional[List[str]]) Resizer step-specific override for PNR_CORNERS.
    "PL_TARGET_DENSITY_PCT": None,  # (Optional[decimal.Decimal]) The desired placement density of cells. If not specified, the value will be e...
    "PL_SKIP_INITIAL_PLACEMENT": False,  # (bool) Specifies whether the placer should run initial placement or not.
    "PL_WIRE_LENGTH_COEF": 0.25,  # (decimal.Decimal) Global placement initial wirelength coefficient. Decreasing the variable will...
    "PL_MIN_PHI_COEFFICIENT": None,  # (Optional[decimal.Decimal]) Sets a lower bound on the µ_k variable in the GPL algorithm. Useful if global...
    "PL_MAX_PHI_COEFFICIENT": None,  # (Optional[decimal.Decimal]) Sets a upper bound on the µ_k variable in the GPL algorithm. Useful if global...
    "FP_CORE_UTIL": 50,  # (decimal.Decimal) The core utilization percentage.
    "GPL_CELL_PADDING": None,  # (int) Cell padding value (in sites) for global placement. The number will be intege...
    "PL_KEEP_RESIZE_BELOW_OVERFLOW": None,  # (Optional[decimal.Decimal]) Only applicable when PL_TIMING_DRIVEN is enabled. When the overflow is below ...
    "IO_PIN_PLACEMENT_MODE": 'matching',  # (Literal['matching', 'annealing', 'random_equidistant']) Decides the mode of the random IO placement option.
    "IO_PIN_ORDER_CFG": None,  # (Optional[librelane.common.types.Path]) Path to a custom pin configuration file.
    "FP_DEF_TEMPLATE": None,  # (Optional[librelane.common.types.Path]) Points to the DEF file to be used as a template.
}
```

#### `OpenROAD.GlobalRouting`
The initial phase of routing. Given a detailed-placed ODB file, this
phase starts assigning coarse-grained routing "regions" for each net so they
may be later connected to wires.

Estimated capacitance and resistance values are much more accurate for
global routing.

* **Inputs**: `odb` (OpenDB Database, `.odb`)
* **Outputs**: `odb` (OpenDB Database, `.odb`), `def` (Design Exchange Format, `.def`)
* **Precursor Steps (Dependencies)**:
* Requires `odb` (OpenDB Database, `.odb`), produced by: `OpenROAD.Floorplan`, `Odb.ApplyDEFTemplate` (or other steps in Odb, OpenROAD)
* **Relevant Step-Specific Config**: `CLOCK_WIRE_RC_LAYERS`, `DEDUPLICATE_CORNERS`, `DIODE_PADDING`, `DPL_CELL_PADDING`, `GRT_ADJUSTMENT`, `GRT_ALLOW_CONGESTION`, `GRT_ANTENNA_REPAIR_DIODE_ONLY`, `GRT_ANTENNA_REPAIR_ITERS`, `GRT_ANTENNA_REPAIR_JUMPER_ONLY`, `GRT_ANTENNA_REPAIR_MARGIN`, `GRT_LAYER_ADJUSTMENTS`, `GRT_MACRO_EXTENSION`, `GRT_OVERFLOW_ITERS`, `LAYERS_RC`, `PDN_CONNECT_MACROS_TO_GRID`, `PDN_ENABLE_GLOBAL_CONNECTIONS`, `PDN_MACRO_CONNECTIONS`, `PL_MAX_DISPLACEMENT_X`, `PL_MAX_DISPLACEMENT_Y`, `PL_OPTIMIZE_MIRRORING`, `PNR_CORNERS`, `PNR_SDC_FILE`, `RT_CLOCK_MAX_LAYER`, `RT_CLOCK_MIN_LAYER`, `SET_RC_VERBOSE`, `SIGNAL_WIRE_RC_LAYERS`, `STA_EXTRA_CORNER_TCL_FILE`, `VIAS_R`

##### Flow Python Integration
```python
# Add this step to your flow's sequential Steps list:
class CustomRun(SequentialFlow):
    Steps = [
        ...  # prior steps
        OpenROAD.GlobalRouting,
        ...  # subsequent steps
    ]
```
##### Configuration Example
```python
# Configure behavior in the flow's config dict:
config = {
    "PNR_CORNERS": None,  # (Optional[List[str]]) A list of fully-qualified IPVT corners to use during PnR. If unspecified, the...
    "SET_RC_VERBOSE": False,  # (bool) If set to true, set_rc commands are echoed. Quite noisy, but may be useful fo...
    "LAYERS_RC": None,  # (Optional[Dict[str, Dict[str, Dict[str, decimal.Decimal]]]]) Used during PNR steps, Specific custom resistance and capacitance values for ...
    "VIAS_R": None,  # (Optional[Dict[str, Dict[str, Dict[str, decimal.Decimal]]]]) Used during PNR steps, Specific custom resistance values for via layers. For ...
    "SIGNAL_WIRE_RC_LAYERS": None,  # (Optional[List[str]]) Sets estimated signal wire RC values to the average of these layers'. If you ...
    "CLOCK_WIRE_RC_LAYERS": None,  # (Optional[List[str]]) Sets estimated clock wire RC values to the average of these layers'. If you p...
    "PDN_CONNECT_MACROS_TO_GRID": True,  # (bool) Enables the connection of macros to the top level power grid.
    "PDN_MACRO_CONNECTIONS": None,  # (Optional[List[str]]) Specifies explicit power connections of internal macros to the top level powe...
    "PDN_ENABLE_GLOBAL_CONNECTIONS": True,  # (bool) Enables the creation of global connections in PDN generation.
    "PNR_SDC_FILE": None,  # (Optional[librelane.common.types.Path]) Specifies the SDC file used during all implementation (PnR) steps
    "STA_EXTRA_CORNER_TCL_FILE": None,  # (Optional[librelane.common.types.Path]) Experimental: specifies a additional configuration .tcl file to be called dur...
    "DEDUPLICATE_CORNERS": False,  # (bool) Cull duplicate IPVT corners during PNR, i.e. corners that share the same set ...
    "RT_CLOCK_MIN_LAYER": None,  # (Optional[str]) The name of lowest layer to be used in routing the clock net.
    "RT_CLOCK_MAX_LAYER": None,  # (Optional[str]) The name of highest layer to be used in routing the clock net.
    "GRT_ADJUSTMENT": 0.3,  # (decimal.Decimal) Reduction in the routing capacity of the edges between the cells in the globa...
    "GRT_MACRO_EXTENSION": 0,  # (int) Sets the number of GCells added to the blockages boundaries from macros. A GC...
    "GRT_LAYER_ADJUSTMENTS": None,  # (List[decimal.Decimal]) Layer-specific reductions in the routing capacity of the edges between the ce...
    "DIODE_PADDING": None,  # (Optional[int]) Diode cell padding; increases the width of diode cells during placement checks..
    "GRT_ALLOW_CONGESTION": False,  # (bool) Allow congestion during global routing
    "GRT_ANTENNA_REPAIR_ITERS": 3,  # (int) The maximum number of iterations for global antenna repairs.
    "GRT_OVERFLOW_ITERS": 50,  # (int) The maximum number of iterations waiting for the overflow to reach the desire...
    "GRT_ANTENNA_REPAIR_MARGIN": 10,  # (int) The margin to over fix antenna violations.
    "GRT_ANTENNA_REPAIR_JUMPER_ONLY": False,  # (bool) Only use jumpers to fix antenna violations. Cannot be used in conjunction wit...
    "GRT_ANTENNA_REPAIR_DIODE_ONLY": False,  # (bool) Only use antenna diodes to fix antenna violations. Cannot be used in conjunct...
    "PL_OPTIMIZE_MIRRORING": True,  # (bool) Specifies whether or not to run an optimize_mirroring pass whenever detailed ...
    "PL_MAX_DISPLACEMENT_X": 500,  # (int) Specifies how far an instance can be moved along the X-axis when finding a si...
    "PL_MAX_DISPLACEMENT_Y": 100,  # (int) Specifies how far an instance can be moved along the Y-axis when finding a si...
    "DPL_CELL_PADDING": None,  # (int) Cell padding value (in sites) for detailed placement. The number will be inte...
}
```

#### `OpenROAD.IOPlacement`
Places I/O pins on a floor-planned ODB file using OpenROAD's built-in placer.

If ``IO_PIN_ORDER_CFG`` is not ``None``, this step is skipped (for
compatibility with OpenLane.)

* **Inputs**: `odb` (OpenDB Database, `.odb`)
* **Outputs**: `odb` (OpenDB Database, `.odb`), `def` (Design Exchange Format, `.def`), `sdc` (Design Constraints, `.sdc`), `nl` (Verilog Netlist, `.nl.v`), `pnl` (Powered Verilog Netlist, `.pnl.v`)
* **Precursor Steps (Dependencies)**:
* Requires `odb` (OpenDB Database, `.odb`), produced by: `OpenROAD.Floorplan`, `Odb.ApplyDEFTemplate` (or other steps in Odb, OpenROAD)
* **Relevant Step-Specific Config**: `CLOCK_WIRE_RC_LAYERS`, `DEDUPLICATE_CORNERS`, `FP_DEF_TEMPLATE`, `IO_EXCLUDE_PIN_REGION`, `IO_PIN_CORNER_AVOIDANCE`, `IO_PIN_H_EXTENSION`, `IO_PIN_H_LAYER`, `IO_PIN_H_LENGTH`, `IO_PIN_H_THICKNESS_MULT`, `IO_PIN_MIN_DISTANCE`, `IO_PIN_MIN_DISTANCE_IN_TRACKS`, `IO_PIN_ORDER_CFG`, `IO_PIN_PLACEMENT_MODE`, `IO_PIN_V_EXTENSION`, `IO_PIN_V_LAYER`, `IO_PIN_V_LENGTH`, `IO_PIN_V_THICKNESS_MULT`, `LAYERS_RC`, `PDN_CONNECT_MACROS_TO_GRID`, `PDN_ENABLE_GLOBAL_CONNECTIONS`, `PDN_MACRO_CONNECTIONS`, `PNR_CORNERS`, `PNR_SDC_FILE`, `SET_RC_VERBOSE`, `SIGNAL_WIRE_RC_LAYERS`, `STA_EXTRA_CORNER_TCL_FILE`, `VIAS_R`

##### Flow Python Integration
```python
# Add this step to your flow's sequential Steps list:
class CustomRun(SequentialFlow):
    Steps = [
        ...  # prior steps
        OpenROAD.IOPlacement,
        ...  # subsequent steps
    ]
```
##### Configuration Example
```python
# Configure behavior in the flow's config dict:
config = {
    "PNR_CORNERS": None,  # (Optional[List[str]]) A list of fully-qualified IPVT corners to use during PnR. If unspecified, the...
    "SET_RC_VERBOSE": False,  # (bool) If set to true, set_rc commands are echoed. Quite noisy, but may be useful fo...
    "LAYERS_RC": None,  # (Optional[Dict[str, Dict[str, Dict[str, decimal.Decimal]]]]) Used during PNR steps, Specific custom resistance and capacitance values for ...
    "VIAS_R": None,  # (Optional[Dict[str, Dict[str, Dict[str, decimal.Decimal]]]]) Used during PNR steps, Specific custom resistance values for via layers. For ...
    "SIGNAL_WIRE_RC_LAYERS": None,  # (Optional[List[str]]) Sets estimated signal wire RC values to the average of these layers'. If you ...
    "CLOCK_WIRE_RC_LAYERS": None,  # (Optional[List[str]]) Sets estimated clock wire RC values to the average of these layers'. If you p...
    "PDN_CONNECT_MACROS_TO_GRID": True,  # (bool) Enables the connection of macros to the top level power grid.
    "PDN_MACRO_CONNECTIONS": None,  # (Optional[List[str]]) Specifies explicit power connections of internal macros to the top level powe...
    "PDN_ENABLE_GLOBAL_CONNECTIONS": True,  # (bool) Enables the creation of global connections in PDN generation.
    "PNR_SDC_FILE": None,  # (Optional[librelane.common.types.Path]) Specifies the SDC file used during all implementation (PnR) steps
    "STA_EXTRA_CORNER_TCL_FILE": None,  # (Optional[librelane.common.types.Path]) Experimental: specifies a additional configuration .tcl file to be called dur...
    "DEDUPLICATE_CORNERS": False,  # (bool) Cull duplicate IPVT corners during PNR, i.e. corners that share the same set ...
    "IO_PIN_H_LAYER": None,  # (str) The metal layer on which to place horizontally-aligned (long side parallel wi...
    "IO_PIN_V_LAYER": None,  # (str) The metal layer on which to place vertically-aligned (long side perpendicular...
    "IO_PIN_V_EXTENSION": 0,  # (decimal.Decimal) Extends the vertical io pins outside of the die by the specified units.
    "IO_PIN_H_EXTENSION": 0,  # (decimal.Decimal) Extends the horizontal io pins outside of the die by the specified units.
    "IO_PIN_V_THICKNESS_MULT": 2,  # (decimal.Decimal) A multiplier for vertical pin thickness. Base thickness is the pins layer min...
    "IO_PIN_H_THICKNESS_MULT": 2,  # (decimal.Decimal) A multiplier for horizontal pin thickness. Base thickness is the pins layer m...
    "IO_PIN_V_LENGTH": None,  # (Optional[decimal.Decimal]) The length of the pins with a north or south orientation. If unspecified by a...
    "IO_PIN_H_LENGTH": None,  # (Optional[decimal.Decimal]) The length of the pins with an east or west orientation. If unspecified by a ...
    "IO_PIN_CORNER_AVOIDANCE": None,  # (Optional[decimal.Decimal]) The distance from each corner within which pin placement should be avoided.
    "IO_PIN_PLACEMENT_MODE": 'matching',  # (Literal['matching', 'annealing', 'random_equidistant']) Decides the mode of the random IO placement option.
    "IO_PIN_MIN_DISTANCE": None,  # (Optional[decimal.Decimal]) The minimum distance between two pins. The unit is microns or routing tracks,...
    "IO_PIN_MIN_DISTANCE_IN_TRACKS": None,  # (Optional[bool]) Setting this variable to true allows IO_PIN_MIN_DISTANCE to be set in number ...
    "IO_PIN_ORDER_CFG": None,  # (Optional[librelane.common.types.Path]) Path to a custom pin configuration file.
    "IO_EXCLUDE_PIN_REGION": None,  # (Optional[List[str]]) List of regions where pins cannot be placed. The regions are strings in the f...
    "FP_DEF_TEMPLATE": None,  # (Optional[librelane.common.types.Path]) Points to the DEF file to be used as a template.
}
```

#### `OpenROAD.IRDropReport`
Performs static IR-drop analysis on the power distribution network. For power
nets, this constitutes a decrease in voltage, and for ground nets, it constitutes
an increase in voltage.

* **Inputs**: `odb` (OpenDB Database, `.odb`), `spef` (Standard Parasitics Extraction Format, `.spef`)
* **Outputs**: None
* **Precursor Steps (Dependencies)**:
* Requires `odb` (OpenDB Database, `.odb`), produced by: `OpenROAD.Floorplan`, `Odb.ApplyDEFTemplate` (or other steps in Odb, OpenROAD)
* Requires `spef` (Standard Parasitics Extraction Format, `.spef`), produced by: `OpenROAD.RCX`
* **Relevant Step-Specific Config**: `CLOCK_WIRE_RC_LAYERS`, `DEDUPLICATE_CORNERS`, `LAYERS_RC`, `PDN_CONNECT_MACROS_TO_GRID`, `PDN_ENABLE_GLOBAL_CONNECTIONS`, `PDN_MACRO_CONNECTIONS`, `PNR_CORNERS`, `PNR_SDC_FILE`, `SET_RC_VERBOSE`, `SIGNAL_WIRE_RC_LAYERS`, `STA_EXTRA_CORNER_TCL_FILE`, `VIAS_R`, `VSRC_LOC_FILES`

##### Flow Python Integration
```python
# Add this step to your flow's sequential Steps list:
class CustomRun(SequentialFlow):
    Steps = [
        ...  # prior steps
        OpenROAD.IRDropReport,
        ...  # subsequent steps
    ]
```
##### Configuration Example
```python
# Configure behavior in the flow's config dict:
config = {
    "PNR_CORNERS": None,  # (Optional[List[str]]) A list of fully-qualified IPVT corners to use during PnR. If unspecified, the...
    "SET_RC_VERBOSE": False,  # (bool) If set to true, set_rc commands are echoed. Quite noisy, but may be useful fo...
    "LAYERS_RC": None,  # (Optional[Dict[str, Dict[str, Dict[str, decimal.Decimal]]]]) Used during PNR steps, Specific custom resistance and capacitance values for ...
    "VIAS_R": None,  # (Optional[Dict[str, Dict[str, Dict[str, decimal.Decimal]]]]) Used during PNR steps, Specific custom resistance values for via layers. For ...
    "SIGNAL_WIRE_RC_LAYERS": None,  # (Optional[List[str]]) Sets estimated signal wire RC values to the average of these layers'. If you ...
    "CLOCK_WIRE_RC_LAYERS": None,  # (Optional[List[str]]) Sets estimated clock wire RC values to the average of these layers'. If you p...
    "PDN_CONNECT_MACROS_TO_GRID": True,  # (bool) Enables the connection of macros to the top level power grid.
    "PDN_MACRO_CONNECTIONS": None,  # (Optional[List[str]]) Specifies explicit power connections of internal macros to the top level powe...
    "PDN_ENABLE_GLOBAL_CONNECTIONS": True,  # (bool) Enables the creation of global connections in PDN generation.
    "PNR_SDC_FILE": None,  # (Optional[librelane.common.types.Path]) Specifies the SDC file used during all implementation (PnR) steps
    "STA_EXTRA_CORNER_TCL_FILE": None,  # (Optional[librelane.common.types.Path]) Experimental: specifies a additional configuration .tcl file to be called dur...
    "DEDUPLICATE_CORNERS": False,  # (bool) Cull duplicate IPVT corners during PNR, i.e. corners that share the same set ...
    "VSRC_LOC_FILES": None,  # (Optional[Dict[str, librelane.common.types.Path]]) Map of power and ground nets to OpenROAD PSM location files. See [this](https...
}
```

#### `OpenROAD.LayoutSTA`
Performs `Static Timing Analysis <https://en.wikipedia.org/wiki/Static_timing_analysis>`_
using OpenROAD on the ODB layout in its current state.

* **Inputs**: `odb` (OpenDB Database, `.odb`)
* **Outputs**: None
* **Precursor Steps (Dependencies)**:
* Requires `odb` (OpenDB Database, `.odb`), produced by: `OpenROAD.Floorplan`, `Odb.ApplyDEFTemplate` (or other steps in Odb, OpenROAD)
* **Relevant Step-Specific Config**: `CLOCK_WIRE_RC_LAYERS`, `DEDUPLICATE_CORNERS`, `LAYERS_RC`, `PDN_CONNECT_MACROS_TO_GRID`, `PDN_ENABLE_GLOBAL_CONNECTIONS`, `PDN_MACRO_CONNECTIONS`, `PNR_CORNERS`, `PNR_SDC_FILE`, `SET_RC_VERBOSE`, `SIGNAL_WIRE_RC_LAYERS`, `STA_EXTRA_CORNER_TCL_FILE`, `VIAS_R`

##### Flow Python Integration
```python
# Add this step to your flow's sequential Steps list:
class CustomRun(SequentialFlow):
    Steps = [
        ...  # prior steps
        OpenROAD.LayoutSTA,
        ...  # subsequent steps
    ]
```
##### Configuration Example
```python
# Configure behavior in the flow's config dict:
config = {
    "PNR_CORNERS": None,  # (Optional[List[str]]) A list of fully-qualified IPVT corners to use during PnR. If unspecified, the...
    "SET_RC_VERBOSE": False,  # (bool) If set to true, set_rc commands are echoed. Quite noisy, but may be useful fo...
    "LAYERS_RC": None,  # (Optional[Dict[str, Dict[str, Dict[str, decimal.Decimal]]]]) Used during PNR steps, Specific custom resistance and capacitance values for ...
    "VIAS_R": None,  # (Optional[Dict[str, Dict[str, Dict[str, decimal.Decimal]]]]) Used during PNR steps, Specific custom resistance values for via layers. For ...
    "SIGNAL_WIRE_RC_LAYERS": None,  # (Optional[List[str]]) Sets estimated signal wire RC values to the average of these layers'. If you ...
    "CLOCK_WIRE_RC_LAYERS": None,  # (Optional[List[str]]) Sets estimated clock wire RC values to the average of these layers'. If you p...
    "PDN_CONNECT_MACROS_TO_GRID": True,  # (bool) Enables the connection of macros to the top level power grid.
    "PDN_MACRO_CONNECTIONS": None,  # (Optional[List[str]]) Specifies explicit power connections of internal macros to the top level powe...
    "PDN_ENABLE_GLOBAL_CONNECTIONS": True,  # (bool) Enables the creation of global connections in PDN generation.
    "PNR_SDC_FILE": None,  # (Optional[librelane.common.types.Path]) Specifies the SDC file used during all implementation (PnR) steps
    "STA_EXTRA_CORNER_TCL_FILE": None,  # (Optional[librelane.common.types.Path]) Experimental: specifies a additional configuration .tcl file to be called dur...
    "DEDUPLICATE_CORNERS": False,  # (bool) Cull duplicate IPVT corners during PNR, i.e. corners that share the same set ...
}
```

#### `OpenROAD.OpenGUI`
Opens the ODB view in the OpenROAD GUI. Useful to inspect some parameters,
such as routing density, timing paths, clock tree and whatnot.
The LIBs are loaded by default and the SPEFs if available.

* **Inputs**: `odb` (OpenDB Database, `.odb`), `spef` (Standard Parasitics Extraction Format, `.spef`)
* **Outputs**: None
* **Precursor Steps (Dependencies)**:
* Requires `odb` (OpenDB Database, `.odb`), produced by: `OpenROAD.Floorplan`, `Odb.ApplyDEFTemplate` (or other steps in Odb, OpenROAD)
* Requires `spef` (Standard Parasitics Extraction Format, `.spef`), produced by: `OpenROAD.RCX`
* **Relevant Step-Specific Config**: `CLOCK_WIRE_RC_LAYERS`, `DEDUPLICATE_CORNERS`, `LAYERS_RC`, `PDN_CONNECT_MACROS_TO_GRID`, `PDN_ENABLE_GLOBAL_CONNECTIONS`, `PDN_MACRO_CONNECTIONS`, `PNR_CORNERS`, `PNR_SDC_FILE`, `SET_RC_VERBOSE`, `SIGNAL_WIRE_RC_LAYERS`, `STA_EXTRA_CORNER_TCL_FILE`, `VIAS_R`

##### Flow Python Integration
```python
# Add this step to your flow's sequential Steps list:
class CustomRun(SequentialFlow):
    Steps = [
        ...  # prior steps
        OpenROAD.OpenGUI,
        ...  # subsequent steps
    ]
```
##### Configuration Example
```python
# Configure behavior in the flow's config dict:
config = {
    "PNR_CORNERS": None,  # (Optional[List[str]]) A list of fully-qualified IPVT corners to use during PnR. If unspecified, the...
    "SET_RC_VERBOSE": False,  # (bool) If set to true, set_rc commands are echoed. Quite noisy, but may be useful fo...
    "LAYERS_RC": None,  # (Optional[Dict[str, Dict[str, Dict[str, decimal.Decimal]]]]) Used during PNR steps, Specific custom resistance and capacitance values for ...
    "VIAS_R": None,  # (Optional[Dict[str, Dict[str, Dict[str, decimal.Decimal]]]]) Used during PNR steps, Specific custom resistance values for via layers. For ...
    "SIGNAL_WIRE_RC_LAYERS": None,  # (Optional[List[str]]) Sets estimated signal wire RC values to the average of these layers'. If you ...
    "CLOCK_WIRE_RC_LAYERS": None,  # (Optional[List[str]]) Sets estimated clock wire RC values to the average of these layers'. If you p...
    "PDN_CONNECT_MACROS_TO_GRID": True,  # (bool) Enables the connection of macros to the top level power grid.
    "PDN_MACRO_CONNECTIONS": None,  # (Optional[List[str]]) Specifies explicit power connections of internal macros to the top level powe...
    "PDN_ENABLE_GLOBAL_CONNECTIONS": True,  # (bool) Enables the creation of global connections in PDN generation.
    "PNR_SDC_FILE": None,  # (Optional[librelane.common.types.Path]) Specifies the SDC file used during all implementation (PnR) steps
    "STA_EXTRA_CORNER_TCL_FILE": None,  # (Optional[librelane.common.types.Path]) Experimental: specifies a additional configuration .tcl file to be called dur...
    "DEDUPLICATE_CORNERS": False,  # (bool) Cull duplicate IPVT corners during PNR, i.e. corners that share the same set ...
}
```

#### `OpenROAD.PadRing`
Assembles a pad ring on a floor-planned ODB file using OpenROAD's built-in pad placer.

* **Inputs**: `odb` (OpenDB Database, `.odb`)
* **Outputs**: `odb` (OpenDB Database, `.odb`), `def` (Design Exchange Format, `.def`), `sdc` (Design Constraints, `.sdc`), `nl` (Verilog Netlist, `.nl.v`), `pnl` (Powered Verilog Netlist, `.pnl.v`)
* **Precursor Steps (Dependencies)**:
* Requires `odb` (OpenDB Database, `.odb`), produced by: `OpenROAD.Floorplan`, `Odb.ApplyDEFTemplate` (or other steps in Odb, OpenROAD)
* **Relevant Step-Specific Config**: `CLOCK_WIRE_RC_LAYERS`, `DEDUPLICATE_CORNERS`, `LAYERS_RC`, `PAD_CFG`, `PAD_EAST`, `PAD_NORTH`, `PAD_SOUTH`, `PAD_WEST`, `PDN_CONNECT_MACROS_TO_GRID`, `PDN_CONNECT_MACROS_TO_GRID`, `PDN_ENABLE_GLOBAL_CONNECTIONS`, `PDN_ENABLE_GLOBAL_CONNECTIONS`, `PDN_MACRO_CONNECTIONS`, `PDN_MACRO_CONNECTIONS`, `PNR_CORNERS`, `PNR_SDC_FILE`, `SET_RC_VERBOSE`, `SIGNAL_WIRE_RC_LAYERS`, `STA_EXTRA_CORNER_TCL_FILE`, `VIAS_R`

##### Flow Python Integration
```python
# Add this step to your flow's sequential Steps list:
class CustomRun(SequentialFlow):
    Steps = [
        ...  # prior steps
        OpenROAD.PadRing,
        ...  # subsequent steps
    ]
```
##### Configuration Example
```python
# Configure behavior in the flow's config dict:
config = {
    "PNR_CORNERS": None,  # (Optional[List[str]]) A list of fully-qualified IPVT corners to use during PnR. If unspecified, the...
    "SET_RC_VERBOSE": False,  # (bool) If set to true, set_rc commands are echoed. Quite noisy, but may be useful fo...
    "LAYERS_RC": None,  # (Optional[Dict[str, Dict[str, Dict[str, decimal.Decimal]]]]) Used during PNR steps, Specific custom resistance and capacitance values for ...
    "VIAS_R": None,  # (Optional[Dict[str, Dict[str, Dict[str, decimal.Decimal]]]]) Used during PNR steps, Specific custom resistance values for via layers. For ...
    "SIGNAL_WIRE_RC_LAYERS": None,  # (Optional[List[str]]) Sets estimated signal wire RC values to the average of these layers'. If you ...
    "CLOCK_WIRE_RC_LAYERS": None,  # (Optional[List[str]]) Sets estimated clock wire RC values to the average of these layers'. If you p...
    "PDN_CONNECT_MACROS_TO_GRID": True,  # (bool) Enables the connection of macros to the top level power grid.
    "PDN_MACRO_CONNECTIONS": None,  # (Optional[List[str]]) Specifies explicit power connections of internal macros to the top level powe...
    "PDN_ENABLE_GLOBAL_CONNECTIONS": True,  # (bool) Enables the creation of global connections in PDN generation.
    "PNR_SDC_FILE": None,  # (Optional[librelane.common.types.Path]) Specifies the SDC file used during all implementation (PnR) steps
    "STA_EXTRA_CORNER_TCL_FILE": None,  # (Optional[librelane.common.types.Path]) Experimental: specifies a additional configuration .tcl file to be called dur...
    "DEDUPLICATE_CORNERS": False,  # (bool) Cull duplicate IPVT corners during PNR, i.e. corners that share the same set ...
    "PDN_CONNECT_MACROS_TO_GRID": True,  # (bool) Enables the connection of macros to the top level power grid.
    "PDN_MACRO_CONNECTIONS": None,  # (Optional[List[str]]) Specifies explicit power connections of internal macros to the top level powe...
    "PDN_ENABLE_GLOBAL_CONNECTIONS": True,  # (bool) Enables the creation of global connections in PDN generation.
    "PAD_CFG": None,  # (Optional[librelane.common.types.Path]) A custom pad configuration file. If not provided, the default pad config will...
    "PAD_SOUTH": None,  # (Optional[List[str]]) The pad instance names for the south pad row.
    "PAD_EAST": None,  # (Optional[List[str]]) The pad instance names for the east pad row.
    "PAD_NORTH": None,  # (Optional[List[str]]) The pad instance names for the north pad row.
    "PAD_WEST": None,  # (Optional[List[str]]) The pad instance names for the west pad row.
}
```

#### `OpenROAD.RCX`
This extracts `parasitic <https://en.wikipedia.org/wiki/Parasitic_element_(electrical_networks)>`_
electrical values from a detailed-placed circuit. These can be used to create
basically the highest accurate STA possible for a given design.

* **Inputs**: `def` (Design Exchange Format, `.def`)
* **Outputs**: `spef` (Standard Parasitics Extraction Format, `.spef`)
* **Precursor Steps (Dependencies)**:
* Requires `def` (Design Exchange Format, `.def`), produced by: `OpenROAD.Floorplan`, `Odb.ApplyDEFTemplate` (or other steps in Odb, OpenROAD)
* **Relevant Step-Specific Config**: `CLOCK_WIRE_RC_LAYERS`, `DEDUPLICATE_CORNERS`, `LAYERS_RC`, `PDN_CONNECT_MACROS_TO_GRID`, `PDN_ENABLE_GLOBAL_CONNECTIONS`, `PDN_MACRO_CONNECTIONS`, `PNR_CORNERS`, `PNR_SDC_FILE`, `RCX_MERGE_VIA_WIRE_RES`, `RCX_RULESETS`, `RCX_SDC_FILE`, `SET_RC_VERBOSE`, `SIGNAL_WIRE_RC_LAYERS`, `STA_EXTRA_CORNER_TCL_FILE`, `STA_THREADS`, `VIAS_R`

##### Flow Python Integration
```python
# Add this step to your flow's sequential Steps list:
class CustomRun(SequentialFlow):
    Steps = [
        ...  # prior steps
        OpenROAD.RCX,
        ...  # subsequent steps
    ]
```
##### Configuration Example
```python
# Configure behavior in the flow's config dict:
config = {
    "PNR_CORNERS": None,  # (Optional[List[str]]) A list of fully-qualified IPVT corners to use during PnR. If unspecified, the...
    "SET_RC_VERBOSE": False,  # (bool) If set to true, set_rc commands are echoed. Quite noisy, but may be useful fo...
    "LAYERS_RC": None,  # (Optional[Dict[str, Dict[str, Dict[str, decimal.Decimal]]]]) Used during PNR steps, Specific custom resistance and capacitance values for ...
    "VIAS_R": None,  # (Optional[Dict[str, Dict[str, Dict[str, decimal.Decimal]]]]) Used during PNR steps, Specific custom resistance values for via layers. For ...
    "SIGNAL_WIRE_RC_LAYERS": None,  # (Optional[List[str]]) Sets estimated signal wire RC values to the average of these layers'. If you ...
    "CLOCK_WIRE_RC_LAYERS": None,  # (Optional[List[str]]) Sets estimated clock wire RC values to the average of these layers'. If you p...
    "PDN_CONNECT_MACROS_TO_GRID": True,  # (bool) Enables the connection of macros to the top level power grid.
    "PDN_MACRO_CONNECTIONS": None,  # (Optional[List[str]]) Specifies explicit power connections of internal macros to the top level powe...
    "PDN_ENABLE_GLOBAL_CONNECTIONS": True,  # (bool) Enables the creation of global connections in PDN generation.
    "PNR_SDC_FILE": None,  # (Optional[librelane.common.types.Path]) Specifies the SDC file used during all implementation (PnR) steps
    "STA_EXTRA_CORNER_TCL_FILE": None,  # (Optional[librelane.common.types.Path]) Experimental: specifies a additional configuration .tcl file to be called dur...
    "DEDUPLICATE_CORNERS": False,  # (bool) Cull duplicate IPVT corners during PNR, i.e. corners that share the same set ...
    "RCX_MERGE_VIA_WIRE_RES": True,  # (bool) If enabled, the via and wire resistances will be merged.
    "RCX_SDC_FILE": None,  # (Optional[librelane.common.types.Path]) Specifies SDC file to be used for RCX-based STA, which can be different from ...
    "RCX_RULESETS": None,  # (Dict[str, librelane.common.types.Path]) Map of corner patterns to OpenRCX extraction rules.
    "STA_THREADS": None,  # (Optional[int]) The maximum number of STA corners to run in parallel. If unset, this will be ...
}
```

#### `OpenROAD.RepairAntennas`
Applies `antenna effect <https://en.wikipedia.org/wiki/Antenna_effect>`_
mitigations using global-routing information, then re-runs detailed placement
and global routing to legalize any inserted diodes.

An antenna check is once again performed, updating the
``route__antenna_violation__count`` metric.

* **Inputs**: `odb` (OpenDB Database, `.odb`)
* **Outputs**: `def` (Design Exchange Format, `.def`), `odb` (OpenDB Database, `.odb`)
* **Precursor Steps (Dependencies)**:
* Requires `odb` (OpenDB Database, `.odb`), produced by: `OpenROAD.Floorplan`, `Odb.ApplyDEFTemplate` (or other steps in Odb, OpenROAD)
* **Relevant Step-Specific Config**: `CLOCK_WIRE_RC_LAYERS`, `DEDUPLICATE_CORNERS`, `DIODE_PADDING`, `DPL_CELL_PADDING`, `GRT_ADJUSTMENT`, `GRT_ALLOW_CONGESTION`, `GRT_ANTENNA_REPAIR_DIODE_ONLY`, `GRT_ANTENNA_REPAIR_ITERS`, `GRT_ANTENNA_REPAIR_JUMPER_ONLY`, `GRT_ANTENNA_REPAIR_MARGIN`, `GRT_LAYER_ADJUSTMENTS`, `GRT_MACRO_EXTENSION`, `GRT_OVERFLOW_ITERS`, `LAYERS_RC`, `PDN_CONNECT_MACROS_TO_GRID`, `PDN_ENABLE_GLOBAL_CONNECTIONS`, `PDN_MACRO_CONNECTIONS`, `PL_MAX_DISPLACEMENT_X`, `PL_MAX_DISPLACEMENT_Y`, `PL_OPTIMIZE_MIRRORING`, `PNR_CORNERS`, `PNR_SDC_FILE`, `RT_CLOCK_MAX_LAYER`, `RT_CLOCK_MIN_LAYER`, `SET_RC_VERBOSE`, `SIGNAL_WIRE_RC_LAYERS`, `STA_EXTRA_CORNER_TCL_FILE`, `VIAS_R`

##### Flow Python Integration
```python
# Add this step to your flow's sequential Steps list:
class CustomRun(SequentialFlow):
    Steps = [
        ...  # prior steps
        OpenROAD.RepairAntennas,
        ...  # subsequent steps
    ]
```
##### Configuration Example
```python
# Configure behavior in the flow's config dict:
config = {
    "PNR_CORNERS": None,  # (Optional[List[str]]) A list of fully-qualified IPVT corners to use during PnR. If unspecified, the...
    "SET_RC_VERBOSE": False,  # (bool) If set to true, set_rc commands are echoed. Quite noisy, but may be useful fo...
    "LAYERS_RC": None,  # (Optional[Dict[str, Dict[str, Dict[str, decimal.Decimal]]]]) Used during PNR steps, Specific custom resistance and capacitance values for ...
    "VIAS_R": None,  # (Optional[Dict[str, Dict[str, Dict[str, decimal.Decimal]]]]) Used during PNR steps, Specific custom resistance values for via layers. For ...
    "SIGNAL_WIRE_RC_LAYERS": None,  # (Optional[List[str]]) Sets estimated signal wire RC values to the average of these layers'. If you ...
    "CLOCK_WIRE_RC_LAYERS": None,  # (Optional[List[str]]) Sets estimated clock wire RC values to the average of these layers'. If you p...
    "PDN_CONNECT_MACROS_TO_GRID": True,  # (bool) Enables the connection of macros to the top level power grid.
    "PDN_MACRO_CONNECTIONS": None,  # (Optional[List[str]]) Specifies explicit power connections of internal macros to the top level powe...
    "PDN_ENABLE_GLOBAL_CONNECTIONS": True,  # (bool) Enables the creation of global connections in PDN generation.
    "PNR_SDC_FILE": None,  # (Optional[librelane.common.types.Path]) Specifies the SDC file used during all implementation (PnR) steps
    "STA_EXTRA_CORNER_TCL_FILE": None,  # (Optional[librelane.common.types.Path]) Experimental: specifies a additional configuration .tcl file to be called dur...
    "DEDUPLICATE_CORNERS": False,  # (bool) Cull duplicate IPVT corners during PNR, i.e. corners that share the same set ...
    "RT_CLOCK_MIN_LAYER": None,  # (Optional[str]) The name of lowest layer to be used in routing the clock net.
    "RT_CLOCK_MAX_LAYER": None,  # (Optional[str]) The name of highest layer to be used in routing the clock net.
    "GRT_ADJUSTMENT": 0.3,  # (decimal.Decimal) Reduction in the routing capacity of the edges between the cells in the globa...
    "GRT_MACRO_EXTENSION": 0,  # (int) Sets the number of GCells added to the blockages boundaries from macros. A GC...
    "GRT_LAYER_ADJUSTMENTS": None,  # (List[decimal.Decimal]) Layer-specific reductions in the routing capacity of the edges between the ce...
    "DIODE_PADDING": None,  # (Optional[int]) Diode cell padding; increases the width of diode cells during placement checks..
    "GRT_ALLOW_CONGESTION": False,  # (bool) Allow congestion during global routing
    "GRT_ANTENNA_REPAIR_ITERS": 3,  # (int) The maximum number of iterations for global antenna repairs.
    "GRT_OVERFLOW_ITERS": 50,  # (int) The maximum number of iterations waiting for the overflow to reach the desire...
    "GRT_ANTENNA_REPAIR_MARGIN": 10,  # (int) The margin to over fix antenna violations.
    "GRT_ANTENNA_REPAIR_JUMPER_ONLY": False,  # (bool) Only use jumpers to fix antenna violations. Cannot be used in conjunction wit...
    "GRT_ANTENNA_REPAIR_DIODE_ONLY": False,  # (bool) Only use antenna diodes to fix antenna violations. Cannot be used in conjunct...
    "PL_OPTIMIZE_MIRRORING": True,  # (bool) Specifies whether or not to run an optimize_mirroring pass whenever detailed ...
    "PL_MAX_DISPLACEMENT_X": 500,  # (int) Specifies how far an instance can be moved along the X-axis when finding a si...
    "PL_MAX_DISPLACEMENT_Y": 100,  # (int) Specifies how far an instance can be moved along the Y-axis when finding a si...
    "DPL_CELL_PADDING": None,  # (int) Cell padding value (in sites) for detailed placement. The number will be inte...
}
```

#### `OpenROAD.RepairDesign`
This is identical to OpenROAD.RepairDesignPostGPL. It is retained for backwards compatibility.

* **Inputs**: `odb` (OpenDB Database, `.odb`)
* **Outputs**: `odb` (OpenDB Database, `.odb`), `def` (Design Exchange Format, `.def`), `sdc` (Design Constraints, `.sdc`), `nl` (Verilog Netlist, `.nl.v`), `pnl` (Powered Verilog Netlist, `.pnl.v`)
* **Precursor Steps (Dependencies)**:
* Requires `odb` (OpenDB Database, `.odb`), produced by: `OpenROAD.Floorplan`, `Odb.ApplyDEFTemplate` (or other steps in Odb, OpenROAD)
* **Relevant Step-Specific Config**: `CLOCK_WIRE_RC_LAYERS`, `DEDUPLICATE_CORNERS`, `DESIGN_REPAIR_BUFFER_INPUT_PORTS`, `DESIGN_REPAIR_BUFFER_OUTPUT_PORTS`, `DESIGN_REPAIR_MAX_CAP_PCT`, `DESIGN_REPAIR_MAX_SLEW_PCT`, `DESIGN_REPAIR_MAX_WIRE_LENGTH`, `DESIGN_REPAIR_REMOVE_BUFFERS`, `DESIGN_REPAIR_TIE_FANOUT`, `DESIGN_REPAIR_TIE_SEPARATION`, `DIODE_PADDING`, `DPL_CELL_PADDING`, `GRT_ADJUSTMENT`, `GRT_ALLOW_CONGESTION`, `GRT_ANTENNA_REPAIR_DIODE_ONLY`, `GRT_ANTENNA_REPAIR_ITERS`, `GRT_ANTENNA_REPAIR_JUMPER_ONLY`, `GRT_ANTENNA_REPAIR_MARGIN`, `GRT_LAYER_ADJUSTMENTS`, `GRT_MACRO_EXTENSION`, `GRT_OVERFLOW_ITERS`, `LAYERS_RC`, `PDN_CONNECT_MACROS_TO_GRID`, `PDN_ENABLE_GLOBAL_CONNECTIONS`, `PDN_MACRO_CONNECTIONS`, `PL_MAX_DISPLACEMENT_X`, `PL_MAX_DISPLACEMENT_Y`, `PL_OPTIMIZE_MIRRORING`, `PNR_CORNERS`, `PNR_SDC_FILE`, `RSZ_CORNERS`, `RSZ_DONT_TOUCH_LIST`, `RSZ_DONT_TOUCH_RX`, `RT_CLOCK_MAX_LAYER`, `RT_CLOCK_MIN_LAYER`, `SET_RC_VERBOSE`, `SIGNAL_WIRE_RC_LAYERS`, `STA_EXTRA_CORNER_TCL_FILE`, `VIAS_R`

##### Flow Python Integration
```python
# Add this step to your flow's sequential Steps list:
class CustomRun(SequentialFlow):
    Steps = [
        ...  # prior steps
        OpenROAD.RepairDesign,
        ...  # subsequent steps
    ]
```
##### Configuration Example
```python
# Configure behavior in the flow's config dict:
config = {
    "PNR_CORNERS": None,  # (Optional[List[str]]) A list of fully-qualified IPVT corners to use during PnR. If unspecified, the...
    "SET_RC_VERBOSE": False,  # (bool) If set to true, set_rc commands are echoed. Quite noisy, but may be useful fo...
    "LAYERS_RC": None,  # (Optional[Dict[str, Dict[str, Dict[str, decimal.Decimal]]]]) Used during PNR steps, Specific custom resistance and capacitance values for ...
    "VIAS_R": None,  # (Optional[Dict[str, Dict[str, Dict[str, decimal.Decimal]]]]) Used during PNR steps, Specific custom resistance values for via layers. For ...
    "SIGNAL_WIRE_RC_LAYERS": None,  # (Optional[List[str]]) Sets estimated signal wire RC values to the average of these layers'. If you ...
    "CLOCK_WIRE_RC_LAYERS": None,  # (Optional[List[str]]) Sets estimated clock wire RC values to the average of these layers'. If you p...
    "PDN_CONNECT_MACROS_TO_GRID": True,  # (bool) Enables the connection of macros to the top level power grid.
    "PDN_MACRO_CONNECTIONS": None,  # (Optional[List[str]]) Specifies explicit power connections of internal macros to the top level powe...
    "PDN_ENABLE_GLOBAL_CONNECTIONS": True,  # (bool) Enables the creation of global connections in PDN generation.
    "PNR_SDC_FILE": None,  # (Optional[librelane.common.types.Path]) Specifies the SDC file used during all implementation (PnR) steps
    "STA_EXTRA_CORNER_TCL_FILE": None,  # (Optional[librelane.common.types.Path]) Experimental: specifies a additional configuration .tcl file to be called dur...
    "DEDUPLICATE_CORNERS": False,  # (bool) Cull duplicate IPVT corners during PNR, i.e. corners that share the same set ...
    "RT_CLOCK_MIN_LAYER": None,  # (Optional[str]) The name of lowest layer to be used in routing the clock net.
    "RT_CLOCK_MAX_LAYER": None,  # (Optional[str]) The name of highest layer to be used in routing the clock net.
    "GRT_ADJUSTMENT": 0.3,  # (decimal.Decimal) Reduction in the routing capacity of the edges between the cells in the globa...
    "GRT_MACRO_EXTENSION": 0,  # (int) Sets the number of GCells added to the blockages boundaries from macros. A GC...
    "GRT_LAYER_ADJUSTMENTS": None,  # (List[decimal.Decimal]) Layer-specific reductions in the routing capacity of the edges between the ce...
    "DIODE_PADDING": None,  # (Optional[int]) Diode cell padding; increases the width of diode cells during placement checks..
    "GRT_ALLOW_CONGESTION": False,  # (bool) Allow congestion during global routing
    "GRT_ANTENNA_REPAIR_ITERS": 3,  # (int) The maximum number of iterations for global antenna repairs.
    "GRT_OVERFLOW_ITERS": 50,  # (int) The maximum number of iterations waiting for the overflow to reach the desire...
    "GRT_ANTENNA_REPAIR_MARGIN": 10,  # (int) The margin to over fix antenna violations.
    "GRT_ANTENNA_REPAIR_JUMPER_ONLY": False,  # (bool) Only use jumpers to fix antenna violations. Cannot be used in conjunction wit...
    "GRT_ANTENNA_REPAIR_DIODE_ONLY": False,  # (bool) Only use antenna diodes to fix antenna violations. Cannot be used in conjunct...
    "PL_OPTIMIZE_MIRRORING": True,  # (bool) Specifies whether or not to run an optimize_mirroring pass whenever detailed ...
    "PL_MAX_DISPLACEMENT_X": 500,  # (int) Specifies how far an instance can be moved along the X-axis when finding a si...
    "PL_MAX_DISPLACEMENT_Y": 100,  # (int) Specifies how far an instance can be moved along the Y-axis when finding a si...
    "DPL_CELL_PADDING": None,  # (int) Cell padding value (in sites) for detailed placement. The number will be inte...
    "RSZ_DONT_TOUCH_RX": '\$^',  # (str) A single regular expression designating nets or instances as "don't touch" by...
    "RSZ_DONT_TOUCH_LIST": None,  # (Optional[List[str]]) A list of nets and instances as "don't touch" by design repairs or resizer op...
    "RSZ_CORNERS": None,  # (Optional[List[str]]) Resizer step-specific override for PNR_CORNERS.
    "DESIGN_REPAIR_BUFFER_INPUT_PORTS": True,  # (bool) Specifies whether or not to insert buffers on input ports when design repairs...
    "DESIGN_REPAIR_BUFFER_OUTPUT_PORTS": True,  # (bool) Specifies whether or not to insert buffers on output ports when design repair...
    "DESIGN_REPAIR_TIE_FANOUT": True,  # (bool) Specifies whether or not to repair tie cells fanout when design repairs are run.
    "DESIGN_REPAIR_TIE_SEPARATION": False,  # (bool) Allows tie separation when performing design repairs.
    "DESIGN_REPAIR_MAX_WIRE_LENGTH": 0,  # (decimal.Decimal) Specifies the maximum wire length cap used by resizer to insert buffers durin...
    "DESIGN_REPAIR_MAX_SLEW_PCT": 20,  # (decimal.Decimal) Specifies a margin for the slews during design repair.
    "DESIGN_REPAIR_MAX_CAP_PCT": 20,  # (decimal.Decimal) Specifies a margin for the capacitances during design repair.
    "DESIGN_REPAIR_REMOVE_BUFFERS": False,  # (bool) Invokes OpenROAD's remove_buffers command to remove buffers from synthesis, w...
}
```

#### `OpenROAD.RepairDesignPostGPL`
Runs a number of design "repairs" on a global-placed ODB file.

* **Inputs**: `odb` (OpenDB Database, `.odb`)
* **Outputs**: `odb` (OpenDB Database, `.odb`), `def` (Design Exchange Format, `.def`), `sdc` (Design Constraints, `.sdc`), `nl` (Verilog Netlist, `.nl.v`), `pnl` (Powered Verilog Netlist, `.pnl.v`)
* **Precursor Steps (Dependencies)**:
* Requires `odb` (OpenDB Database, `.odb`), produced by: `OpenROAD.Floorplan`, `Odb.ApplyDEFTemplate` (or other steps in Odb, OpenROAD)
* **Relevant Step-Specific Config**: `CLOCK_WIRE_RC_LAYERS`, `DEDUPLICATE_CORNERS`, `DESIGN_REPAIR_BUFFER_INPUT_PORTS`, `DESIGN_REPAIR_BUFFER_OUTPUT_PORTS`, `DESIGN_REPAIR_MAX_CAP_PCT`, `DESIGN_REPAIR_MAX_SLEW_PCT`, `DESIGN_REPAIR_MAX_WIRE_LENGTH`, `DESIGN_REPAIR_REMOVE_BUFFERS`, `DESIGN_REPAIR_TIE_FANOUT`, `DESIGN_REPAIR_TIE_SEPARATION`, `DIODE_PADDING`, `DPL_CELL_PADDING`, `GRT_ADJUSTMENT`, `GRT_ALLOW_CONGESTION`, `GRT_ANTENNA_REPAIR_DIODE_ONLY`, `GRT_ANTENNA_REPAIR_ITERS`, `GRT_ANTENNA_REPAIR_JUMPER_ONLY`, `GRT_ANTENNA_REPAIR_MARGIN`, `GRT_LAYER_ADJUSTMENTS`, `GRT_MACRO_EXTENSION`, `GRT_OVERFLOW_ITERS`, `LAYERS_RC`, `PDN_CONNECT_MACROS_TO_GRID`, `PDN_ENABLE_GLOBAL_CONNECTIONS`, `PDN_MACRO_CONNECTIONS`, `PL_MAX_DISPLACEMENT_X`, `PL_MAX_DISPLACEMENT_Y`, `PL_OPTIMIZE_MIRRORING`, `PNR_CORNERS`, `PNR_SDC_FILE`, `RSZ_CORNERS`, `RSZ_DONT_TOUCH_LIST`, `RSZ_DONT_TOUCH_RX`, `RT_CLOCK_MAX_LAYER`, `RT_CLOCK_MIN_LAYER`, `SET_RC_VERBOSE`, `SIGNAL_WIRE_RC_LAYERS`, `STA_EXTRA_CORNER_TCL_FILE`, `VIAS_R`

##### Flow Python Integration
```python
# Add this step to your flow's sequential Steps list:
class CustomRun(SequentialFlow):
    Steps = [
        ...  # prior steps
        OpenROAD.RepairDesignPostGPL,
        ...  # subsequent steps
    ]
```
##### Configuration Example
```python
# Configure behavior in the flow's config dict:
config = {
    "PNR_CORNERS": None,  # (Optional[List[str]]) A list of fully-qualified IPVT corners to use during PnR. If unspecified, the...
    "SET_RC_VERBOSE": False,  # (bool) If set to true, set_rc commands are echoed. Quite noisy, but may be useful fo...
    "LAYERS_RC": None,  # (Optional[Dict[str, Dict[str, Dict[str, decimal.Decimal]]]]) Used during PNR steps, Specific custom resistance and capacitance values for ...
    "VIAS_R": None,  # (Optional[Dict[str, Dict[str, Dict[str, decimal.Decimal]]]]) Used during PNR steps, Specific custom resistance values for via layers. For ...
    "SIGNAL_WIRE_RC_LAYERS": None,  # (Optional[List[str]]) Sets estimated signal wire RC values to the average of these layers'. If you ...
    "CLOCK_WIRE_RC_LAYERS": None,  # (Optional[List[str]]) Sets estimated clock wire RC values to the average of these layers'. If you p...
    "PDN_CONNECT_MACROS_TO_GRID": True,  # (bool) Enables the connection of macros to the top level power grid.
    "PDN_MACRO_CONNECTIONS": None,  # (Optional[List[str]]) Specifies explicit power connections of internal macros to the top level powe...
    "PDN_ENABLE_GLOBAL_CONNECTIONS": True,  # (bool) Enables the creation of global connections in PDN generation.
    "PNR_SDC_FILE": None,  # (Optional[librelane.common.types.Path]) Specifies the SDC file used during all implementation (PnR) steps
    "STA_EXTRA_CORNER_TCL_FILE": None,  # (Optional[librelane.common.types.Path]) Experimental: specifies a additional configuration .tcl file to be called dur...
    "DEDUPLICATE_CORNERS": False,  # (bool) Cull duplicate IPVT corners during PNR, i.e. corners that share the same set ...
    "RT_CLOCK_MIN_LAYER": None,  # (Optional[str]) The name of lowest layer to be used in routing the clock net.
    "RT_CLOCK_MAX_LAYER": None,  # (Optional[str]) The name of highest layer to be used in routing the clock net.
    "GRT_ADJUSTMENT": 0.3,  # (decimal.Decimal) Reduction in the routing capacity of the edges between the cells in the globa...
    "GRT_MACRO_EXTENSION": 0,  # (int) Sets the number of GCells added to the blockages boundaries from macros. A GC...
    "GRT_LAYER_ADJUSTMENTS": None,  # (List[decimal.Decimal]) Layer-specific reductions in the routing capacity of the edges between the ce...
    "DIODE_PADDING": None,  # (Optional[int]) Diode cell padding; increases the width of diode cells during placement checks..
    "GRT_ALLOW_CONGESTION": False,  # (bool) Allow congestion during global routing
    "GRT_ANTENNA_REPAIR_ITERS": 3,  # (int) The maximum number of iterations for global antenna repairs.
    "GRT_OVERFLOW_ITERS": 50,  # (int) The maximum number of iterations waiting for the overflow to reach the desire...
    "GRT_ANTENNA_REPAIR_MARGIN": 10,  # (int) The margin to over fix antenna violations.
    "GRT_ANTENNA_REPAIR_JUMPER_ONLY": False,  # (bool) Only use jumpers to fix antenna violations. Cannot be used in conjunction wit...
    "GRT_ANTENNA_REPAIR_DIODE_ONLY": False,  # (bool) Only use antenna diodes to fix antenna violations. Cannot be used in conjunct...
    "PL_OPTIMIZE_MIRRORING": True,  # (bool) Specifies whether or not to run an optimize_mirroring pass whenever detailed ...
    "PL_MAX_DISPLACEMENT_X": 500,  # (int) Specifies how far an instance can be moved along the X-axis when finding a si...
    "PL_MAX_DISPLACEMENT_Y": 100,  # (int) Specifies how far an instance can be moved along the Y-axis when finding a si...
    "DPL_CELL_PADDING": None,  # (int) Cell padding value (in sites) for detailed placement. The number will be inte...
    "RSZ_DONT_TOUCH_RX": '\$^',  # (str) A single regular expression designating nets or instances as "don't touch" by...
    "RSZ_DONT_TOUCH_LIST": None,  # (Optional[List[str]]) A list of nets and instances as "don't touch" by design repairs or resizer op...
    "RSZ_CORNERS": None,  # (Optional[List[str]]) Resizer step-specific override for PNR_CORNERS.
    "DESIGN_REPAIR_BUFFER_INPUT_PORTS": True,  # (bool) Specifies whether or not to insert buffers on input ports when design repairs...
    "DESIGN_REPAIR_BUFFER_OUTPUT_PORTS": True,  # (bool) Specifies whether or not to insert buffers on output ports when design repair...
    "DESIGN_REPAIR_TIE_FANOUT": True,  # (bool) Specifies whether or not to repair tie cells fanout when design repairs are run.
    "DESIGN_REPAIR_TIE_SEPARATION": False,  # (bool) Allows tie separation when performing design repairs.
    "DESIGN_REPAIR_MAX_WIRE_LENGTH": 0,  # (decimal.Decimal) Specifies the maximum wire length cap used by resizer to insert buffers durin...
    "DESIGN_REPAIR_MAX_SLEW_PCT": 20,  # (decimal.Decimal) Specifies a margin for the slews during design repair.
    "DESIGN_REPAIR_MAX_CAP_PCT": 20,  # (decimal.Decimal) Specifies a margin for the capacitances during design repair.
    "DESIGN_REPAIR_REMOVE_BUFFERS": False,  # (bool) Invokes OpenROAD's remove_buffers command to remove buffers from synthesis, w...
}
```

#### `OpenROAD.RepairDesignPostGRT`
Runs a number of design "repairs" on a global-routed ODB file.

* **Inputs**: `odb` (OpenDB Database, `.odb`)
* **Outputs**: `odb` (OpenDB Database, `.odb`), `def` (Design Exchange Format, `.def`), `sdc` (Design Constraints, `.sdc`), `nl` (Verilog Netlist, `.nl.v`), `pnl` (Powered Verilog Netlist, `.pnl.v`)
* **Precursor Steps (Dependencies)**:
* Requires `odb` (OpenDB Database, `.odb`), produced by: `OpenROAD.Floorplan`, `Odb.ApplyDEFTemplate` (or other steps in Odb, OpenROAD)
* **Relevant Step-Specific Config**: `CLOCK_WIRE_RC_LAYERS`, `DEDUPLICATE_CORNERS`, `DIODE_PADDING`, `DPL_CELL_PADDING`, `GRT_ADJUSTMENT`, `GRT_ALLOW_CONGESTION`, `GRT_ANTENNA_REPAIR_DIODE_ONLY`, `GRT_ANTENNA_REPAIR_ITERS`, `GRT_ANTENNA_REPAIR_JUMPER_ONLY`, `GRT_ANTENNA_REPAIR_MARGIN`, `GRT_DESIGN_REPAIR_MAX_CAP_PCT`, `GRT_DESIGN_REPAIR_MAX_SLEW_PCT`, `GRT_DESIGN_REPAIR_MAX_WIRE_LENGTH`, `GRT_DESIGN_REPAIR_RUN_GRT`, `GRT_LAYER_ADJUSTMENTS`, `GRT_MACRO_EXTENSION`, `GRT_OVERFLOW_ITERS`, `LAYERS_RC`, `PDN_CONNECT_MACROS_TO_GRID`, `PDN_ENABLE_GLOBAL_CONNECTIONS`, `PDN_MACRO_CONNECTIONS`, `PL_MAX_DISPLACEMENT_X`, `PL_MAX_DISPLACEMENT_Y`, `PL_OPTIMIZE_MIRRORING`, `PNR_CORNERS`, `PNR_SDC_FILE`, `RSZ_CORNERS`, `RSZ_DONT_TOUCH_LIST`, `RSZ_DONT_TOUCH_RX`, `RT_CLOCK_MAX_LAYER`, `RT_CLOCK_MIN_LAYER`, `SET_RC_VERBOSE`, `SIGNAL_WIRE_RC_LAYERS`, `STA_EXTRA_CORNER_TCL_FILE`, `VIAS_R`

##### Flow Python Integration
```python
# Add this step to your flow's sequential Steps list:
class CustomRun(SequentialFlow):
    Steps = [
        ...  # prior steps
        OpenROAD.RepairDesignPostGRT,
        ...  # subsequent steps
    ]
```
##### Configuration Example
```python
# Configure behavior in the flow's config dict:
config = {
    "PNR_CORNERS": None,  # (Optional[List[str]]) A list of fully-qualified IPVT corners to use during PnR. If unspecified, the...
    "SET_RC_VERBOSE": False,  # (bool) If set to true, set_rc commands are echoed. Quite noisy, but may be useful fo...
    "LAYERS_RC": None,  # (Optional[Dict[str, Dict[str, Dict[str, decimal.Decimal]]]]) Used during PNR steps, Specific custom resistance and capacitance values for ...
    "VIAS_R": None,  # (Optional[Dict[str, Dict[str, Dict[str, decimal.Decimal]]]]) Used during PNR steps, Specific custom resistance values for via layers. For ...
    "SIGNAL_WIRE_RC_LAYERS": None,  # (Optional[List[str]]) Sets estimated signal wire RC values to the average of these layers'. If you ...
    "CLOCK_WIRE_RC_LAYERS": None,  # (Optional[List[str]]) Sets estimated clock wire RC values to the average of these layers'. If you p...
    "PDN_CONNECT_MACROS_TO_GRID": True,  # (bool) Enables the connection of macros to the top level power grid.
    "PDN_MACRO_CONNECTIONS": None,  # (Optional[List[str]]) Specifies explicit power connections of internal macros to the top level powe...
    "PDN_ENABLE_GLOBAL_CONNECTIONS": True,  # (bool) Enables the creation of global connections in PDN generation.
    "PNR_SDC_FILE": None,  # (Optional[librelane.common.types.Path]) Specifies the SDC file used during all implementation (PnR) steps
    "STA_EXTRA_CORNER_TCL_FILE": None,  # (Optional[librelane.common.types.Path]) Experimental: specifies a additional configuration .tcl file to be called dur...
    "DEDUPLICATE_CORNERS": False,  # (bool) Cull duplicate IPVT corners during PNR, i.e. corners that share the same set ...
    "RT_CLOCK_MIN_LAYER": None,  # (Optional[str]) The name of lowest layer to be used in routing the clock net.
    "RT_CLOCK_MAX_LAYER": None,  # (Optional[str]) The name of highest layer to be used in routing the clock net.
    "GRT_ADJUSTMENT": 0.3,  # (decimal.Decimal) Reduction in the routing capacity of the edges between the cells in the globa...
    "GRT_MACRO_EXTENSION": 0,  # (int) Sets the number of GCells added to the blockages boundaries from macros. A GC...
    "GRT_LAYER_ADJUSTMENTS": None,  # (List[decimal.Decimal]) Layer-specific reductions in the routing capacity of the edges between the ce...
    "DIODE_PADDING": None,  # (Optional[int]) Diode cell padding; increases the width of diode cells during placement checks..
    "GRT_ALLOW_CONGESTION": False,  # (bool) Allow congestion during global routing
    "GRT_ANTENNA_REPAIR_ITERS": 3,  # (int) The maximum number of iterations for global antenna repairs.
    "GRT_OVERFLOW_ITERS": 50,  # (int) The maximum number of iterations waiting for the overflow to reach the desire...
    "GRT_ANTENNA_REPAIR_MARGIN": 10,  # (int) The margin to over fix antenna violations.
    "GRT_ANTENNA_REPAIR_JUMPER_ONLY": False,  # (bool) Only use jumpers to fix antenna violations. Cannot be used in conjunction wit...
    "GRT_ANTENNA_REPAIR_DIODE_ONLY": False,  # (bool) Only use antenna diodes to fix antenna violations. Cannot be used in conjunct...
    "PL_OPTIMIZE_MIRRORING": True,  # (bool) Specifies whether or not to run an optimize_mirroring pass whenever detailed ...
    "PL_MAX_DISPLACEMENT_X": 500,  # (int) Specifies how far an instance can be moved along the X-axis when finding a si...
    "PL_MAX_DISPLACEMENT_Y": 100,  # (int) Specifies how far an instance can be moved along the Y-axis when finding a si...
    "DPL_CELL_PADDING": None,  # (int) Cell padding value (in sites) for detailed placement. The number will be inte...
    "RSZ_DONT_TOUCH_RX": '\$^',  # (str) A single regular expression designating nets or instances as "don't touch" by...
    "RSZ_DONT_TOUCH_LIST": None,  # (Optional[List[str]]) A list of nets and instances as "don't touch" by design repairs or resizer op...
    "RSZ_CORNERS": None,  # (Optional[List[str]]) Resizer step-specific override for PNR_CORNERS.
    "GRT_DESIGN_REPAIR_RUN_GRT": True,  # (bool) Enables running GRT before and after running resizer
    "GRT_DESIGN_REPAIR_MAX_WIRE_LENGTH": 0,  # (decimal.Decimal) Specifies the maximum wire length cap used by resizer to insert buffers durin...
    "GRT_DESIGN_REPAIR_MAX_SLEW_PCT": 10,  # (decimal.Decimal) Specifies a margin for the slews during post-grt design repair.
    "GRT_DESIGN_REPAIR_MAX_CAP_PCT": 10,  # (decimal.Decimal) Specifies a margin for the capacitances during design post-grt repair.
}
```

#### `OpenROAD.ResizerTimingPostCTS`
First attempt to meet timing requirements for a cell based on basic timing
information after clock tree synthesis.

Standard cells may be resized, and buffer cells may be inserted to ensure
that no hold violations exist and no setup violations exist at the current
clock.

* **Inputs**: `odb` (OpenDB Database, `.odb`)
* **Outputs**: `odb` (OpenDB Database, `.odb`), `def` (Design Exchange Format, `.def`), `sdc` (Design Constraints, `.sdc`), `nl` (Verilog Netlist, `.nl.v`), `pnl` (Powered Verilog Netlist, `.pnl.v`)
* **Precursor Steps (Dependencies)**:
* Requires `odb` (OpenDB Database, `.odb`), produced by: `OpenROAD.Floorplan`, `Odb.ApplyDEFTemplate` (or other steps in Odb, OpenROAD)
* **Relevant Step-Specific Config**: `CLOCK_WIRE_RC_LAYERS`, `DEDUPLICATE_CORNERS`, `DIODE_PADDING`, `DPL_CELL_PADDING`, `GRT_ADJUSTMENT`, `GRT_ALLOW_CONGESTION`, `GRT_ANTENNA_REPAIR_DIODE_ONLY`, `GRT_ANTENNA_REPAIR_ITERS`, `GRT_ANTENNA_REPAIR_JUMPER_ONLY`, `GRT_ANTENNA_REPAIR_MARGIN`, `GRT_LAYER_ADJUSTMENTS`, `GRT_MACRO_EXTENSION`, `GRT_OVERFLOW_ITERS`, `LAYERS_RC`, `PDN_CONNECT_MACROS_TO_GRID`, `PDN_ENABLE_GLOBAL_CONNECTIONS`, `PDN_MACRO_CONNECTIONS`, `PL_MAX_DISPLACEMENT_X`, `PL_MAX_DISPLACEMENT_Y`, `PL_OPTIMIZE_MIRRORING`, `PL_RESIZER_ALLOW_SETUP_VIOS`, `PL_RESIZER_FIX_HOLD_FIRST`, `PL_RESIZER_HOLD_MAX_BUFFER_PCT`, `PL_RESIZER_HOLD_MAX_UTIL_PCT`, `PL_RESIZER_HOLD_REPAIR_TNS_PCT`, `PL_RESIZER_HOLD_SLACK_MARGIN`, `PL_RESIZER_SETUP_BUFFERING`, `PL_RESIZER_SETUP_BUFFER_REMOVAL`, `PL_RESIZER_SETUP_GATE_CLONING`, `PL_RESIZER_SETUP_MAX_BUFFER_PCT`, `PL_RESIZER_SETUP_MAX_UTIL_PCT`, `PL_RESIZER_SETUP_REPAIR_TNS_PCT`, `PL_RESIZER_SETUP_SLACK_MARGIN`, `PNR_CORNERS`, `PNR_SDC_FILE`, `RSZ_CORNERS`, `RSZ_DONT_TOUCH_LIST`, `RSZ_DONT_TOUCH_RX`, `RT_CLOCK_MAX_LAYER`, `RT_CLOCK_MIN_LAYER`, `SET_RC_VERBOSE`, `SIGNAL_WIRE_RC_LAYERS`, `STA_EXTRA_CORNER_TCL_FILE`, `VIAS_R`

##### Flow Python Integration
```python
# Add this step to your flow's sequential Steps list:
class CustomRun(SequentialFlow):
    Steps = [
        ...  # prior steps
        OpenROAD.ResizerTimingPostCTS,
        ...  # subsequent steps
    ]
```
##### Configuration Example
```python
# Configure behavior in the flow's config dict:
config = {
    "PNR_CORNERS": None,  # (Optional[List[str]]) A list of fully-qualified IPVT corners to use during PnR. If unspecified, the...
    "SET_RC_VERBOSE": False,  # (bool) If set to true, set_rc commands are echoed. Quite noisy, but may be useful fo...
    "LAYERS_RC": None,  # (Optional[Dict[str, Dict[str, Dict[str, decimal.Decimal]]]]) Used during PNR steps, Specific custom resistance and capacitance values for ...
    "VIAS_R": None,  # (Optional[Dict[str, Dict[str, Dict[str, decimal.Decimal]]]]) Used during PNR steps, Specific custom resistance values for via layers. For ...
    "SIGNAL_WIRE_RC_LAYERS": None,  # (Optional[List[str]]) Sets estimated signal wire RC values to the average of these layers'. If you ...
    "CLOCK_WIRE_RC_LAYERS": None,  # (Optional[List[str]]) Sets estimated clock wire RC values to the average of these layers'. If you p...
    "PDN_CONNECT_MACROS_TO_GRID": True,  # (bool) Enables the connection of macros to the top level power grid.
    "PDN_MACRO_CONNECTIONS": None,  # (Optional[List[str]]) Specifies explicit power connections of internal macros to the top level powe...
    "PDN_ENABLE_GLOBAL_CONNECTIONS": True,  # (bool) Enables the creation of global connections in PDN generation.
    "PNR_SDC_FILE": None,  # (Optional[librelane.common.types.Path]) Specifies the SDC file used during all implementation (PnR) steps
    "STA_EXTRA_CORNER_TCL_FILE": None,  # (Optional[librelane.common.types.Path]) Experimental: specifies a additional configuration .tcl file to be called dur...
    "DEDUPLICATE_CORNERS": False,  # (bool) Cull duplicate IPVT corners during PNR, i.e. corners that share the same set ...
    "RT_CLOCK_MIN_LAYER": None,  # (Optional[str]) The name of lowest layer to be used in routing the clock net.
    "RT_CLOCK_MAX_LAYER": None,  # (Optional[str]) The name of highest layer to be used in routing the clock net.
    "GRT_ADJUSTMENT": 0.3,  # (decimal.Decimal) Reduction in the routing capacity of the edges between the cells in the globa...
    "GRT_MACRO_EXTENSION": 0,  # (int) Sets the number of GCells added to the blockages boundaries from macros. A GC...
    "GRT_LAYER_ADJUSTMENTS": None,  # (List[decimal.Decimal]) Layer-specific reductions in the routing capacity of the edges between the ce...
    "DIODE_PADDING": None,  # (Optional[int]) Diode cell padding; increases the width of diode cells during placement checks..
    "GRT_ALLOW_CONGESTION": False,  # (bool) Allow congestion during global routing
    "GRT_ANTENNA_REPAIR_ITERS": 3,  # (int) The maximum number of iterations for global antenna repairs.
    "GRT_OVERFLOW_ITERS": 50,  # (int) The maximum number of iterations waiting for the overflow to reach the desire...
    "GRT_ANTENNA_REPAIR_MARGIN": 10,  # (int) The margin to over fix antenna violations.
    "GRT_ANTENNA_REPAIR_JUMPER_ONLY": False,  # (bool) Only use jumpers to fix antenna violations. Cannot be used in conjunction wit...
    "GRT_ANTENNA_REPAIR_DIODE_ONLY": False,  # (bool) Only use antenna diodes to fix antenna violations. Cannot be used in conjunct...
    "PL_OPTIMIZE_MIRRORING": True,  # (bool) Specifies whether or not to run an optimize_mirroring pass whenever detailed ...
    "PL_MAX_DISPLACEMENT_X": 500,  # (int) Specifies how far an instance can be moved along the X-axis when finding a si...
    "PL_MAX_DISPLACEMENT_Y": 100,  # (int) Specifies how far an instance can be moved along the Y-axis when finding a si...
    "DPL_CELL_PADDING": None,  # (int) Cell padding value (in sites) for detailed placement. The number will be inte...
    "RSZ_DONT_TOUCH_RX": '\$^',  # (str) A single regular expression designating nets or instances as "don't touch" by...
    "RSZ_DONT_TOUCH_LIST": None,  # (Optional[List[str]]) A list of nets and instances as "don't touch" by design repairs or resizer op...
    "RSZ_CORNERS": None,  # (Optional[List[str]]) Resizer step-specific override for PNR_CORNERS.
    "PL_RESIZER_HOLD_SLACK_MARGIN": 0.1,  # (decimal.Decimal) Specifies a time margin for the slack when fixing hold violations. Normally t...
    "PL_RESIZER_SETUP_SLACK_MARGIN": 0.05,  # (decimal.Decimal) Specifies a time margin for the slack when fixing setup violations.
    "PL_RESIZER_HOLD_MAX_BUFFER_PCT": 50,  # (decimal.Decimal) Specifies a max number of buffers to insert to fix hold violations. This numb...
    "PL_RESIZER_SETUP_MAX_BUFFER_PCT": 50,  # (decimal.Decimal) Specifies a max number of buffers to insert to fix setup violations. This num...
    "PL_RESIZER_ALLOW_SETUP_VIOS": False,  # (bool) Allows the creation of setup violations when fixing hold violations. Setup vi...
    "PL_RESIZER_SETUP_GATE_CLONING": True,  # (bool) Enables gate cloning when attempting to fix setup violations
    "PL_RESIZER_SETUP_BUFFERING": True,  # (bool) Rebuffering and load splitting during setup fixing.
    "PL_RESIZER_SETUP_BUFFER_REMOVAL": True,  # (bool) Buffer removal transform during setup fixing.
    "PL_RESIZER_SETUP_REPAIR_TNS_PCT": None,  # (Optional[decimal.Decimal]) Percentage of violating endpoints to repair during setup fixing.
    "PL_RESIZER_SETUP_MAX_UTIL_PCT": None,  # (Optional[decimal.Decimal]) Defines the percentage of core area used during setup fixing.
    "PL_RESIZER_HOLD_REPAIR_TNS_PCT": None,  # (Optional[decimal.Decimal]) Percentage of violating endpoints to repair during hold fixing.
    "PL_RESIZER_HOLD_MAX_UTIL_PCT": None,  # (Optional[decimal.Decimal]) Defines the percentage of core area used during hold fixing.
    "PL_RESIZER_FIX_HOLD_FIRST": False,  # (bool) Experimental: attempt to fix hold violations before setup violations, which m...
}
```

#### `OpenROAD.ResizerTimingPostGRT`
Second attempt to meet timing requirements for a cell based on timing
information after estimating resistance and capacitance values based on
global routing.

Standard cells may be resized, and buffer cells may be inserted to ensure
that no hold violations exist and no setup violations exist at the current
clock.

* **Inputs**: `odb` (OpenDB Database, `.odb`)
* **Outputs**: `odb` (OpenDB Database, `.odb`), `def` (Design Exchange Format, `.def`), `sdc` (Design Constraints, `.sdc`), `nl` (Verilog Netlist, `.nl.v`), `pnl` (Powered Verilog Netlist, `.pnl.v`)
* **Precursor Steps (Dependencies)**:
* Requires `odb` (OpenDB Database, `.odb`), produced by: `OpenROAD.Floorplan`, `Odb.ApplyDEFTemplate` (or other steps in Odb, OpenROAD)
* **Relevant Step-Specific Config**: `CLOCK_WIRE_RC_LAYERS`, `DEDUPLICATE_CORNERS`, `DIODE_PADDING`, `DPL_CELL_PADDING`, `GRT_ADJUSTMENT`, `GRT_ALLOW_CONGESTION`, `GRT_ANTENNA_REPAIR_DIODE_ONLY`, `GRT_ANTENNA_REPAIR_ITERS`, `GRT_ANTENNA_REPAIR_JUMPER_ONLY`, `GRT_ANTENNA_REPAIR_MARGIN`, `GRT_LAYER_ADJUSTMENTS`, `GRT_MACRO_EXTENSION`, `GRT_OVERFLOW_ITERS`, `GRT_RESIZER_ALLOW_SETUP_VIOS`, `GRT_RESIZER_FIX_HOLD_FIRST`, `GRT_RESIZER_HOLD_MAX_BUFFER_PCT`, `GRT_RESIZER_HOLD_MAX_UTIL_PCT`, `GRT_RESIZER_HOLD_REPAIR_TNS_PCT`, `GRT_RESIZER_HOLD_SLACK_MARGIN`, `GRT_RESIZER_RUN_GRT`, `GRT_RESIZER_SETUP_BUFFERING`, `GRT_RESIZER_SETUP_BUFFER_REMOVAL`, `GRT_RESIZER_SETUP_GATE_CLONING`, `GRT_RESIZER_SETUP_MAX_BUFFER_PCT`, `GRT_RESIZER_SETUP_MAX_UTIL_PCT`, `GRT_RESIZER_SETUP_REPAIR_TNS_PCT`, `GRT_RESIZER_SETUP_SLACK_MARGIN`, `LAYERS_RC`, `PDN_CONNECT_MACROS_TO_GRID`, `PDN_ENABLE_GLOBAL_CONNECTIONS`, `PDN_MACRO_CONNECTIONS`, `PL_MAX_DISPLACEMENT_X`, `PL_MAX_DISPLACEMENT_Y`, `PL_OPTIMIZE_MIRRORING`, `PNR_CORNERS`, `PNR_SDC_FILE`, `RSZ_CORNERS`, `RSZ_DONT_TOUCH_LIST`, `RSZ_DONT_TOUCH_RX`, `RT_CLOCK_MAX_LAYER`, `RT_CLOCK_MIN_LAYER`, `SET_RC_VERBOSE`, `SIGNAL_WIRE_RC_LAYERS`, `STA_EXTRA_CORNER_TCL_FILE`, `VIAS_R`

##### Flow Python Integration
```python
# Add this step to your flow's sequential Steps list:
class CustomRun(SequentialFlow):
    Steps = [
        ...  # prior steps
        OpenROAD.ResizerTimingPostGRT,
        ...  # subsequent steps
    ]
```
##### Configuration Example
```python
# Configure behavior in the flow's config dict:
config = {
    "PNR_CORNERS": None,  # (Optional[List[str]]) A list of fully-qualified IPVT corners to use during PnR. If unspecified, the...
    "SET_RC_VERBOSE": False,  # (bool) If set to true, set_rc commands are echoed. Quite noisy, but may be useful fo...
    "LAYERS_RC": None,  # (Optional[Dict[str, Dict[str, Dict[str, decimal.Decimal]]]]) Used during PNR steps, Specific custom resistance and capacitance values for ...
    "VIAS_R": None,  # (Optional[Dict[str, Dict[str, Dict[str, decimal.Decimal]]]]) Used during PNR steps, Specific custom resistance values for via layers. For ...
    "SIGNAL_WIRE_RC_LAYERS": None,  # (Optional[List[str]]) Sets estimated signal wire RC values to the average of these layers'. If you ...
    "CLOCK_WIRE_RC_LAYERS": None,  # (Optional[List[str]]) Sets estimated clock wire RC values to the average of these layers'. If you p...
    "PDN_CONNECT_MACROS_TO_GRID": True,  # (bool) Enables the connection of macros to the top level power grid.
    "PDN_MACRO_CONNECTIONS": None,  # (Optional[List[str]]) Specifies explicit power connections of internal macros to the top level powe...
    "PDN_ENABLE_GLOBAL_CONNECTIONS": True,  # (bool) Enables the creation of global connections in PDN generation.
    "PNR_SDC_FILE": None,  # (Optional[librelane.common.types.Path]) Specifies the SDC file used during all implementation (PnR) steps
    "STA_EXTRA_CORNER_TCL_FILE": None,  # (Optional[librelane.common.types.Path]) Experimental: specifies a additional configuration .tcl file to be called dur...
    "DEDUPLICATE_CORNERS": False,  # (bool) Cull duplicate IPVT corners during PNR, i.e. corners that share the same set ...
    "RT_CLOCK_MIN_LAYER": None,  # (Optional[str]) The name of lowest layer to be used in routing the clock net.
    "RT_CLOCK_MAX_LAYER": None,  # (Optional[str]) The name of highest layer to be used in routing the clock net.
    "GRT_ADJUSTMENT": 0.3,  # (decimal.Decimal) Reduction in the routing capacity of the edges between the cells in the globa...
    "GRT_MACRO_EXTENSION": 0,  # (int) Sets the number of GCells added to the blockages boundaries from macros. A GC...
    "GRT_LAYER_ADJUSTMENTS": None,  # (List[decimal.Decimal]) Layer-specific reductions in the routing capacity of the edges between the ce...
    "DIODE_PADDING": None,  # (Optional[int]) Diode cell padding; increases the width of diode cells during placement checks..
    "GRT_ALLOW_CONGESTION": False,  # (bool) Allow congestion during global routing
    "GRT_ANTENNA_REPAIR_ITERS": 3,  # (int) The maximum number of iterations for global antenna repairs.
    "GRT_OVERFLOW_ITERS": 50,  # (int) The maximum number of iterations waiting for the overflow to reach the desire...
    "GRT_ANTENNA_REPAIR_MARGIN": 10,  # (int) The margin to over fix antenna violations.
    "GRT_ANTENNA_REPAIR_JUMPER_ONLY": False,  # (bool) Only use jumpers to fix antenna violations. Cannot be used in conjunction wit...
    "GRT_ANTENNA_REPAIR_DIODE_ONLY": False,  # (bool) Only use antenna diodes to fix antenna violations. Cannot be used in conjunct...
    "PL_OPTIMIZE_MIRRORING": True,  # (bool) Specifies whether or not to run an optimize_mirroring pass whenever detailed ...
    "PL_MAX_DISPLACEMENT_X": 500,  # (int) Specifies how far an instance can be moved along the X-axis when finding a si...
    "PL_MAX_DISPLACEMENT_Y": 100,  # (int) Specifies how far an instance can be moved along the Y-axis when finding a si...
    "DPL_CELL_PADDING": None,  # (int) Cell padding value (in sites) for detailed placement. The number will be inte...
    "RSZ_DONT_TOUCH_RX": '\$^',  # (str) A single regular expression designating nets or instances as "don't touch" by...
    "RSZ_DONT_TOUCH_LIST": None,  # (Optional[List[str]]) A list of nets and instances as "don't touch" by design repairs or resizer op...
    "RSZ_CORNERS": None,  # (Optional[List[str]]) Resizer step-specific override for PNR_CORNERS.
    "GRT_RESIZER_HOLD_SLACK_MARGIN": 0.05,  # (decimal.Decimal) Specifies a time margin for the slack when fixing hold violations. Normally t...
    "GRT_RESIZER_SETUP_SLACK_MARGIN": 0.025,  # (decimal.Decimal) Specifies a time margin for the slack when fixing setup violations.
    "GRT_RESIZER_HOLD_MAX_BUFFER_PCT": 50,  # (decimal.Decimal) Specifies a max number of buffers to insert to fix hold violations. This numb...
    "GRT_RESIZER_SETUP_MAX_BUFFER_PCT": 50,  # (decimal.Decimal) Specifies a max number of buffers to insert to fix setup violations. This num...
    "GRT_RESIZER_ALLOW_SETUP_VIOS": False,  # (bool) Allows setup violations when fixing hold.
    "GRT_RESIZER_SETUP_GATE_CLONING": True,  # (bool) Enables gate cloning when attempting to fix setup violations
    "GRT_RESIZER_RUN_GRT": True,  # (bool) Gates running global routing after resizer steps. May be useful to disable fo...
    "GRT_RESIZER_SETUP_BUFFERING": True,  # (bool) Rebuffering and load splitting during setup fixing.
    "GRT_RESIZER_SETUP_BUFFER_REMOVAL": True,  # (bool) Buffer removal transform during setup fixing.
    "GRT_RESIZER_SETUP_REPAIR_TNS_PCT": None,  # (Optional[decimal.Decimal]) Percentage of violating endpoints to repair during setup fixing.
    "GRT_RESIZER_SETUP_MAX_UTIL_PCT": None,  # (Optional[decimal.Decimal]) Defines the percentage of core area used during setup fixing.
    "GRT_RESIZER_HOLD_REPAIR_TNS_PCT": None,  # (Optional[decimal.Decimal]) Percentage of violating endpoints to repair during hold fixing.
    "GRT_RESIZER_HOLD_MAX_UTIL_PCT": None,  # (Optional[decimal.Decimal]) Defines the percentage of core area used during hold fixing.
    "GRT_RESIZER_FIX_HOLD_FIRST": False,  # (bool) Experimental: attempt to fix hold violations before setup violations, which m...
}
```

#### `OpenROAD.STAMidPNR`
Performs `Static Timing Analysis <https://en.wikipedia.org/wiki/Static_timing_analysis>`_
using OpenROAD on an OpenROAD database, mid-PnR, with estimated values for
parasitics.

* **Inputs**: `odb` (OpenDB Database, `.odb`)
* **Outputs**: None
* **Precursor Steps (Dependencies)**:
* Requires `odb` (OpenDB Database, `.odb`), produced by: `OpenROAD.Floorplan`, `Odb.ApplyDEFTemplate` (or other steps in Odb, OpenROAD)
* **Relevant Step-Specific Config**: `CLOCK_WIRE_RC_LAYERS`, `DEDUPLICATE_CORNERS`, `LAYERS_RC`, `PDN_CONNECT_MACROS_TO_GRID`, `PDN_ENABLE_GLOBAL_CONNECTIONS`, `PDN_MACRO_CONNECTIONS`, `PNR_CORNERS`, `PNR_SDC_FILE`, `SET_RC_VERBOSE`, `SIGNAL_WIRE_RC_LAYERS`, `STA_EXTRA_CORNER_TCL_FILE`, `VIAS_R`

##### Flow Python Integration
```python
# Add this step to your flow's sequential Steps list:
class CustomRun(SequentialFlow):
    Steps = [
        ...  # prior steps
        OpenROAD.STAMidPNR,
        ...  # subsequent steps
    ]
```
##### Configuration Example
```python
# Configure behavior in the flow's config dict:
config = {
    "PNR_CORNERS": None,  # (Optional[List[str]]) A list of fully-qualified IPVT corners to use during PnR. If unspecified, the...
    "SET_RC_VERBOSE": False,  # (bool) If set to true, set_rc commands are echoed. Quite noisy, but may be useful fo...
    "LAYERS_RC": None,  # (Optional[Dict[str, Dict[str, Dict[str, decimal.Decimal]]]]) Used during PNR steps, Specific custom resistance and capacitance values for ...
    "VIAS_R": None,  # (Optional[Dict[str, Dict[str, Dict[str, decimal.Decimal]]]]) Used during PNR steps, Specific custom resistance values for via layers. For ...
    "SIGNAL_WIRE_RC_LAYERS": None,  # (Optional[List[str]]) Sets estimated signal wire RC values to the average of these layers'. If you ...
    "CLOCK_WIRE_RC_LAYERS": None,  # (Optional[List[str]]) Sets estimated clock wire RC values to the average of these layers'. If you p...
    "PDN_CONNECT_MACROS_TO_GRID": True,  # (bool) Enables the connection of macros to the top level power grid.
    "PDN_MACRO_CONNECTIONS": None,  # (Optional[List[str]]) Specifies explicit power connections of internal macros to the top level powe...
    "PDN_ENABLE_GLOBAL_CONNECTIONS": True,  # (bool) Enables the creation of global connections in PDN generation.
    "PNR_SDC_FILE": None,  # (Optional[librelane.common.types.Path]) Specifies the SDC file used during all implementation (PnR) steps
    "STA_EXTRA_CORNER_TCL_FILE": None,  # (Optional[librelane.common.types.Path]) Experimental: specifies a additional configuration .tcl file to be called dur...
    "DEDUPLICATE_CORNERS": False,  # (bool) Cull duplicate IPVT corners during PNR, i.e. corners that share the same set ...
}
```

#### `OpenROAD.STAPostPNR`
Performs multi-corner `Static Timing Analysis <https://en.wikipedia.org/wiki/Static_timing_analysis>`_
using OpenSTA on the post-PnR Verilog netlist, with extracted parasitics for
both the top-level module and any associated macros.

During this step, the special variable `OPENLANE_SDC_IDEAL_CLOCKS` is
exposed to SDC files with a value of `0`. We encourage PNR SDC files to use
propagated clocks at this stage based on this variable's existence and value.

* **Inputs**: `nl` (Verilog Netlist, `.nl.v`), `spef` (Standard Parasitics Extraction Format, `.spef`), `odb` (OpenDB Database, `.odb`)
* **Outputs**: `sdf` (Standard Delay Format, `.sdf`), `sdc` (Design Constraints, `.sdc`), `lib` (LIB Timing Library Format, `.lib`)
* **Precursor Steps (Dependencies)**:
* Requires `nl` (Verilog Netlist, `.nl.v`), produced by: `Yosys.Synthesis`, `OpenROAD.Floorplan`, `Odb.DiodesOnPorts` (or other steps in Odb, OpenROAD, Yosys)
* Requires `spef` (Standard Parasitics Extraction Format, `.spef`), produced by: `OpenROAD.RCX`
* Requires `odb` (OpenDB Database, `.odb`), produced by: `OpenROAD.Floorplan`, `Odb.ApplyDEFTemplate` (or other steps in Odb, OpenROAD)
* **Relevant Step-Specific Config**: `CLOCK_WIRE_RC_LAYERS`, `DEDUPLICATE_CORNERS`, `EXTRA_SPEFS`, `LAYERS_RC`, `PDN_CONNECT_MACROS_TO_GRID`, `PDN_ENABLE_GLOBAL_CONNECTIONS`, `PDN_MACRO_CONNECTIONS`, `PNR_CORNERS`, `PNR_SDC_FILE`, `SET_RC_VERBOSE`, `SIGNAL_WIRE_RC_LAYERS`, `SIGNOFF_SDC_FILE`, `STA_EXTRA_CORNER_TCL_FILE`, `STA_MACRO_PRIORITIZE_NL`, `STA_MAX_VIOLATOR_COUNT`, `STA_THREADS`, `VIAS_R`

##### Flow Python Integration
```python
# Add this step to your flow's sequential Steps list:
class CustomRun(SequentialFlow):
    Steps = [
        ...  # prior steps
        OpenROAD.STAPostPNR,
        ...  # subsequent steps
    ]
```
##### Configuration Example
```python
# Configure behavior in the flow's config dict:
config = {
    "PNR_CORNERS": None,  # (Optional[List[str]]) A list of fully-qualified IPVT corners to use during PnR. If unspecified, the...
    "SET_RC_VERBOSE": False,  # (bool) If set to true, set_rc commands are echoed. Quite noisy, but may be useful fo...
    "LAYERS_RC": None,  # (Optional[Dict[str, Dict[str, Dict[str, decimal.Decimal]]]]) Used during PNR steps, Specific custom resistance and capacitance values for ...
    "VIAS_R": None,  # (Optional[Dict[str, Dict[str, Dict[str, decimal.Decimal]]]]) Used during PNR steps, Specific custom resistance values for via layers. For ...
    "SIGNAL_WIRE_RC_LAYERS": None,  # (Optional[List[str]]) Sets estimated signal wire RC values to the average of these layers'. If you ...
    "CLOCK_WIRE_RC_LAYERS": None,  # (Optional[List[str]]) Sets estimated clock wire RC values to the average of these layers'. If you p...
    "PDN_CONNECT_MACROS_TO_GRID": True,  # (bool) Enables the connection of macros to the top level power grid.
    "PDN_MACRO_CONNECTIONS": None,  # (Optional[List[str]]) Specifies explicit power connections of internal macros to the top level powe...
    "PDN_ENABLE_GLOBAL_CONNECTIONS": True,  # (bool) Enables the creation of global connections in PDN generation.
    "PNR_SDC_FILE": None,  # (Optional[librelane.common.types.Path]) Specifies the SDC file used during all implementation (PnR) steps
    "STA_EXTRA_CORNER_TCL_FILE": None,  # (Optional[librelane.common.types.Path]) Experimental: specifies a additional configuration .tcl file to be called dur...
    "DEDUPLICATE_CORNERS": False,  # (bool) Cull duplicate IPVT corners during PNR, i.e. corners that share the same set ...
    "STA_MACRO_PRIORITIZE_NL": True,  # (bool) Prioritize the use of Netlists + SPEF files over LIB files if available for M...
    "STA_MAX_VIOLATOR_COUNT": None,  # (Optional[int]) Maximum number of violators to list in violator_list.rpt
    "EXTRA_SPEFS": None,  # (Optional[List[Union[str, librelane.common.types.Path]]]) A variable that only exists for backwards compatibility with LibreLane <2.0.0...
    "STA_THREADS": None,  # (Optional[int]) The maximum number of STA corners to run in parallel. If unset, this will be ...
    "SIGNOFF_SDC_FILE": None,  # (Optional[librelane.common.types.Path]) Specifies the SDC file for STA during signoff
}
```

#### `OpenROAD.STAPrePNR`
Performs hierarchical `Static Timing Analysis <https://en.wikipedia.org/wiki/Static_timing_analysis>`_
using OpenSTA on the pre-PnR Verilog netlist, with all available timing information
for standard cells and macros for multiple corners.

If timing information is not available for a Macro, the macro in question
will be black-boxed.

During this step, the special variable `OPENLANE_SDC_IDEAL_CLOCKS` is
exposed to SDC files with a value of `1`. We encourage PNR SDC files to use
ideal clocks at this stage based on this variable's existence and value.

* **Inputs**: `nl` (Verilog Netlist, `.nl.v`)
* **Outputs**: `sdf` (Standard Delay Format, `.sdf`), `sdc` (Design Constraints, `.sdc`)
* **Precursor Steps (Dependencies)**:
* Requires `nl` (Verilog Netlist, `.nl.v`), produced by: `Yosys.Synthesis`, `OpenROAD.Floorplan`, `Odb.DiodesOnPorts` (or other steps in Odb, OpenROAD, Yosys)
* **Relevant Step-Specific Config**: `CLOCK_WIRE_RC_LAYERS`, `DEDUPLICATE_CORNERS`, `EXTRA_SPEFS`, `LAYERS_RC`, `PDN_CONNECT_MACROS_TO_GRID`, `PDN_ENABLE_GLOBAL_CONNECTIONS`, `PDN_MACRO_CONNECTIONS`, `PNR_CORNERS`, `PNR_SDC_FILE`, `SET_RC_VERBOSE`, `SIGNAL_WIRE_RC_LAYERS`, `STA_EXTRA_CORNER_TCL_FILE`, `STA_MACRO_PRIORITIZE_NL`, `STA_MAX_VIOLATOR_COUNT`, `STA_THREADS`, `VIAS_R`

##### Flow Python Integration
```python
# Add this step to your flow's sequential Steps list:
class CustomRun(SequentialFlow):
    Steps = [
        ...  # prior steps
        OpenROAD.STAPrePNR,
        ...  # subsequent steps
    ]
```
##### Configuration Example
```python
# Configure behavior in the flow's config dict:
config = {
    "PNR_CORNERS": None,  # (Optional[List[str]]) A list of fully-qualified IPVT corners to use during PnR. If unspecified, the...
    "SET_RC_VERBOSE": False,  # (bool) If set to true, set_rc commands are echoed. Quite noisy, but may be useful fo...
    "LAYERS_RC": None,  # (Optional[Dict[str, Dict[str, Dict[str, decimal.Decimal]]]]) Used during PNR steps, Specific custom resistance and capacitance values for ...
    "VIAS_R": None,  # (Optional[Dict[str, Dict[str, Dict[str, decimal.Decimal]]]]) Used during PNR steps, Specific custom resistance values for via layers. For ...
    "SIGNAL_WIRE_RC_LAYERS": None,  # (Optional[List[str]]) Sets estimated signal wire RC values to the average of these layers'. If you ...
    "CLOCK_WIRE_RC_LAYERS": None,  # (Optional[List[str]]) Sets estimated clock wire RC values to the average of these layers'. If you p...
    "PDN_CONNECT_MACROS_TO_GRID": True,  # (bool) Enables the connection of macros to the top level power grid.
    "PDN_MACRO_CONNECTIONS": None,  # (Optional[List[str]]) Specifies explicit power connections of internal macros to the top level powe...
    "PDN_ENABLE_GLOBAL_CONNECTIONS": True,  # (bool) Enables the creation of global connections in PDN generation.
    "PNR_SDC_FILE": None,  # (Optional[librelane.common.types.Path]) Specifies the SDC file used during all implementation (PnR) steps
    "STA_EXTRA_CORNER_TCL_FILE": None,  # (Optional[librelane.common.types.Path]) Experimental: specifies a additional configuration .tcl file to be called dur...
    "DEDUPLICATE_CORNERS": False,  # (bool) Cull duplicate IPVT corners during PNR, i.e. corners that share the same set ...
    "STA_MACRO_PRIORITIZE_NL": True,  # (bool) Prioritize the use of Netlists + SPEF files over LIB files if available for M...
    "STA_MAX_VIOLATOR_COUNT": None,  # (Optional[int]) Maximum number of violators to list in violator_list.rpt
    "EXTRA_SPEFS": None,  # (Optional[List[Union[str, librelane.common.types.Path]]]) A variable that only exists for backwards compatibility with LibreLane <2.0.0...
    "STA_THREADS": None,  # (Optional[int]) The maximum number of STA corners to run in parallel. If unset, this will be ...
}
```

#### `OpenROAD.TapEndcapInsertion`
Places welltap cells across a floorplan, as well as endcap cells at the
edges of the floorplan.

* **Inputs**: `odb` (OpenDB Database, `.odb`)
* **Outputs**: `odb` (OpenDB Database, `.odb`), `def` (Design Exchange Format, `.def`), `sdc` (Design Constraints, `.sdc`), `nl` (Verilog Netlist, `.nl.v`), `pnl` (Powered Verilog Netlist, `.pnl.v`)
* **Precursor Steps (Dependencies)**:
* Requires `odb` (OpenDB Database, `.odb`), produced by: `OpenROAD.Floorplan`, `Odb.ApplyDEFTemplate` (or other steps in Odb, OpenROAD)
* **Relevant Step-Specific Config**: `CLOCK_WIRE_RC_LAYERS`, `DEDUPLICATE_CORNERS`, `FP_MACRO_HORIZONTAL_HALO`, `FP_MACRO_VERTICAL_HALO`, `FP_TAPCELL_DIST`, `LAYERS_RC`, `PDN_CONNECT_MACROS_TO_GRID`, `PDN_ENABLE_GLOBAL_CONNECTIONS`, `PDN_MACRO_CONNECTIONS`, `PNR_CORNERS`, `PNR_SDC_FILE`, `SET_RC_VERBOSE`, `SIGNAL_WIRE_RC_LAYERS`, `STA_EXTRA_CORNER_TCL_FILE`, `VIAS_R`

##### Flow Python Integration
```python
# Add this step to your flow's sequential Steps list:
class CustomRun(SequentialFlow):
    Steps = [
        ...  # prior steps
        OpenROAD.TapEndcapInsertion,
        ...  # subsequent steps
    ]
```
##### Configuration Example
```python
# Configure behavior in the flow's config dict:
config = {
    "PNR_CORNERS": None,  # (Optional[List[str]]) A list of fully-qualified IPVT corners to use during PnR. If unspecified, the...
    "SET_RC_VERBOSE": False,  # (bool) If set to true, set_rc commands are echoed. Quite noisy, but may be useful fo...
    "LAYERS_RC": None,  # (Optional[Dict[str, Dict[str, Dict[str, decimal.Decimal]]]]) Used during PNR steps, Specific custom resistance and capacitance values for ...
    "VIAS_R": None,  # (Optional[Dict[str, Dict[str, Dict[str, decimal.Decimal]]]]) Used during PNR steps, Specific custom resistance values for via layers. For ...
    "SIGNAL_WIRE_RC_LAYERS": None,  # (Optional[List[str]]) Sets estimated signal wire RC values to the average of these layers'. If you ...
    "CLOCK_WIRE_RC_LAYERS": None,  # (Optional[List[str]]) Sets estimated clock wire RC values to the average of these layers'. If you p...
    "PDN_CONNECT_MACROS_TO_GRID": True,  # (bool) Enables the connection of macros to the top level power grid.
    "PDN_MACRO_CONNECTIONS": None,  # (Optional[List[str]]) Specifies explicit power connections of internal macros to the top level powe...
    "PDN_ENABLE_GLOBAL_CONNECTIONS": True,  # (bool) Enables the creation of global connections in PDN generation.
    "PNR_SDC_FILE": None,  # (Optional[librelane.common.types.Path]) Specifies the SDC file used during all implementation (PnR) steps
    "STA_EXTRA_CORNER_TCL_FILE": None,  # (Optional[librelane.common.types.Path]) Experimental: specifies a additional configuration .tcl file to be called dur...
    "DEDUPLICATE_CORNERS": False,  # (bool) Cull duplicate IPVT corners during PNR, i.e. corners that share the same set ...
    "FP_TAPCELL_DIST": None,  # (Optional[decimal.Decimal]) The distance between tap cell columns. Must be specified if WELLTAP_CELL is s...
    "FP_MACRO_HORIZONTAL_HALO": 10,  # (decimal.Decimal) Specify the horizontal halo size around macros.
    "FP_MACRO_VERTICAL_HALO": 10,  # (decimal.Decimal) Specify the vertical halo size around macros.
}
```

#### `OpenROAD.UnplaceAll`
Sets placement status of *all* instances to NONE.

Useful in flows where a preliminary placement is needed as a pre-requisite
to something else but that placement must be discarded.

* **Inputs**: `odb` (OpenDB Database, `.odb`)
* **Outputs**: `odb` (OpenDB Database, `.odb`), `def` (Design Exchange Format, `.def`), `sdc` (Design Constraints, `.sdc`), `nl` (Verilog Netlist, `.nl.v`), `pnl` (Powered Verilog Netlist, `.pnl.v`)
* **Precursor Steps (Dependencies)**:
* Requires `odb` (OpenDB Database, `.odb`), produced by: `OpenROAD.Floorplan`, `Odb.ApplyDEFTemplate` (or other steps in Odb, OpenROAD)
* **Relevant Step-Specific Config**: `CLOCK_WIRE_RC_LAYERS`, `DEDUPLICATE_CORNERS`, `LAYERS_RC`, `PDN_CONNECT_MACROS_TO_GRID`, `PDN_ENABLE_GLOBAL_CONNECTIONS`, `PDN_MACRO_CONNECTIONS`, `PNR_CORNERS`, `PNR_SDC_FILE`, `SET_RC_VERBOSE`, `SIGNAL_WIRE_RC_LAYERS`, `STA_EXTRA_CORNER_TCL_FILE`, `VIAS_R`

##### Flow Python Integration
```python
# Add this step to your flow's sequential Steps list:
class CustomRun(SequentialFlow):
    Steps = [
        ...  # prior steps
        OpenROAD.UnplaceAll,
        ...  # subsequent steps
    ]
```
##### Configuration Example
```python
# Configure behavior in the flow's config dict:
config = {
    "PNR_CORNERS": None,  # (Optional[List[str]]) A list of fully-qualified IPVT corners to use during PnR. If unspecified, the...
    "SET_RC_VERBOSE": False,  # (bool) If set to true, set_rc commands are echoed. Quite noisy, but may be useful fo...
    "LAYERS_RC": None,  # (Optional[Dict[str, Dict[str, Dict[str, decimal.Decimal]]]]) Used during PNR steps, Specific custom resistance and capacitance values for ...
    "VIAS_R": None,  # (Optional[Dict[str, Dict[str, Dict[str, decimal.Decimal]]]]) Used during PNR steps, Specific custom resistance values for via layers. For ...
    "SIGNAL_WIRE_RC_LAYERS": None,  # (Optional[List[str]]) Sets estimated signal wire RC values to the average of these layers'. If you ...
    "CLOCK_WIRE_RC_LAYERS": None,  # (Optional[List[str]]) Sets estimated clock wire RC values to the average of these layers'. If you p...
    "PDN_CONNECT_MACROS_TO_GRID": True,  # (bool) Enables the connection of macros to the top level power grid.
    "PDN_MACRO_CONNECTIONS": None,  # (Optional[List[str]]) Specifies explicit power connections of internal macros to the top level powe...
    "PDN_ENABLE_GLOBAL_CONNECTIONS": True,  # (bool) Enables the creation of global connections in PDN generation.
    "PNR_SDC_FILE": None,  # (Optional[librelane.common.types.Path]) Specifies the SDC file used during all implementation (PnR) steps
    "STA_EXTRA_CORNER_TCL_FILE": None,  # (Optional[librelane.common.types.Path]) Experimental: specifies a additional configuration .tcl file to be called dur...
    "DEDUPLICATE_CORNERS": False,  # (bool) Cull duplicate IPVT corners during PNR, i.e. corners that share the same set ...
}
```

#### `OpenROAD.WriteCDL`
Write CDL view of an ODB design

* **Inputs**: `odb` (OpenDB Database, `.odb`)
* **Outputs**: `cdl` (Circuit Design Language, `.cdl`)
* **Precursor Steps (Dependencies)**:
* Requires `odb` (OpenDB Database, `.odb`), produced by: `OpenROAD.Floorplan`, `Odb.ApplyDEFTemplate` (or other steps in Odb, OpenROAD)
* **Relevant Step-Specific Config**: `CLOCK_WIRE_RC_LAYERS`, `DEDUPLICATE_CORNERS`, `LAYERS_RC`, `PDN_CONNECT_MACROS_TO_GRID`, `PDN_ENABLE_GLOBAL_CONNECTIONS`, `PDN_MACRO_CONNECTIONS`, `PNR_CORNERS`, `PNR_SDC_FILE`, `SET_RC_VERBOSE`, `SIGNAL_WIRE_RC_LAYERS`, `STA_EXTRA_CORNER_TCL_FILE`, `VIAS_R`

##### Flow Python Integration
```python
# Add this step to your flow's sequential Steps list:
class CustomRun(SequentialFlow):
    Steps = [
        ...  # prior steps
        OpenROAD.WriteCDL,
        ...  # subsequent steps
    ]
```
##### Configuration Example
```python
# Configure behavior in the flow's config dict:
config = {
    "PNR_CORNERS": None,  # (Optional[List[str]]) A list of fully-qualified IPVT corners to use during PnR. If unspecified, the...
    "SET_RC_VERBOSE": False,  # (bool) If set to true, set_rc commands are echoed. Quite noisy, but may be useful fo...
    "LAYERS_RC": None,  # (Optional[Dict[str, Dict[str, Dict[str, decimal.Decimal]]]]) Used during PNR steps, Specific custom resistance and capacitance values for ...
    "VIAS_R": None,  # (Optional[Dict[str, Dict[str, Dict[str, decimal.Decimal]]]]) Used during PNR steps, Specific custom resistance values for via layers. For ...
    "SIGNAL_WIRE_RC_LAYERS": None,  # (Optional[List[str]]) Sets estimated signal wire RC values to the average of these layers'. If you ...
    "CLOCK_WIRE_RC_LAYERS": None,  # (Optional[List[str]]) Sets estimated clock wire RC values to the average of these layers'. If you p...
    "PDN_CONNECT_MACROS_TO_GRID": True,  # (bool) Enables the connection of macros to the top level power grid.
    "PDN_MACRO_CONNECTIONS": None,  # (Optional[List[str]]) Specifies explicit power connections of internal macros to the top level powe...
    "PDN_ENABLE_GLOBAL_CONNECTIONS": True,  # (bool) Enables the creation of global connections in PDN generation.
    "PNR_SDC_FILE": None,  # (Optional[librelane.common.types.Path]) Specifies the SDC file used during all implementation (PnR) steps
    "STA_EXTRA_CORNER_TCL_FILE": None,  # (Optional[librelane.common.types.Path]) Experimental: specifies a additional configuration .tcl file to be called dur...
    "DEDUPLICATE_CORNERS": False,  # (bool) Cull duplicate IPVT corners during PNR, i.e. corners that share the same set ...
}
```

### Verilator Steps

#### `Verilator.Lint`
Lints inputs RTL Verilog files.

The linting is done with the defines for power and ground inputs on, as more
macros are available with powered netlists than unpowered netlists.

* **Inputs**: None
* **Outputs**: None
* **Precursor Steps (Dependencies)**:
None (typically runs at the start of the flow, or only requires static design files)
* **Relevant Step-Specific Config**: `LINTER_DEFINES`, `LINTER_DISABLE_WARNINGS`, `LINTER_DISABLE_WARNINGS_BLACKBOX`, `LINTER_ERROR_ON_LATCH`, `LINTER_ERROR_ON_MULTIDRIVEN`, `LINTER_INCLUDE_PDK_MODELS`, `LINTER_RELATIVE_INCLUDES`, `LINTER_VLT`, `VERILOG_DEFINES`, `VERILOG_FILES`, `VERILOG_INCLUDE_DIRS`, `VERILOG_POWER_DEFINE`

##### Flow Python Integration
```python
# Add this step to your flow's sequential Steps list:
class CustomRun(SequentialFlow):
    Steps = [
        ...  # prior steps
        Verilator.Lint,
        ...  # subsequent steps
    ]
```
##### Configuration Example
```python
# Configure behavior in the flow's config dict:
config = {
    "VERILOG_FILES": None,  # (List[librelane.common.types.Path]) The paths of the design's Verilog files.
    "VERILOG_INCLUDE_DIRS": None,  # (Optional[List[librelane.common.types.Path]]) Specifies the Verilog `include` directories.
    "VERILOG_POWER_DEFINE": 'USE_POWER_PINS',  # (Optional[str]) Specifies the name of the define used to guard power and ground connections i...
    "LINTER_INCLUDE_PDK_MODELS": False,  # (bool) Include Verilog models of the PDK
    "LINTER_RELATIVE_INCLUDES": True,  # (bool) When a file references an include file, resolve the filename relative to the ...
    "LINTER_ERROR_ON_LATCH": True,  # (bool) When a latch is inferred by an `always` block that is not explicitly marked a...
    "LINTER_ERROR_ON_MULTIDRIVEN": True,  # (bool) When a net has multiple drivers, report this as a linter error.
    "VERILOG_DEFINES": None,  # (Optional[List[str]]) Preprocessor defines for input Verilog files
    "LINTER_DEFINES": None,  # (Optional[List[str]]) Linter-specific preprocessor definitions; overrides VERILOG_DEFINES for the l...
    "LINTER_DISABLE_WARNINGS": ['DECLFILENAME', 'EOFNEWLINE'],  # (Optional[List[str]]) Warning codes that are passed to the linter to be disabled.
    "LINTER_DISABLE_WARNINGS_BLACKBOX": ['UNDRIVEN', 'UNUSEDSIGNAL'],  # (Optional[List[str]]) Warning codes that are passed to the linter to be disabled for all blackbox m...
    "LINTER_VLT": None,  # (Optional[librelane.common.types.Path]) Path to a Verilator Configuration format file (`.vlt`) that is passed to the ...
}
```

### Yosys Steps

#### `Yosys.EQY`
Experimental: Uses the `EQY <https://github.com/yosyshq/eqy>`_ utility to
perform an RTL vs. Netlist equivalence check.

Currently, you are expected to provide your own EQY script if you want this
to work properly.

* **Inputs**: `nl` (Verilog Netlist, `.nl.v`)
* **Outputs**: None
* **Precursor Steps (Dependencies)**:
* Requires `nl` (Verilog Netlist, `.nl.v`), produced by: `Yosys.Synthesis`, `OpenROAD.Floorplan`, `Odb.DiodesOnPorts` (or other steps in Odb, OpenROAD, Yosys)
* **Relevant Step-Specific Config**: `EQY_FORCE_ACCEPT_PDK`, `EQY_SCRIPT`, `MACRO_PLACEMENT_CFG`, `SLANG_ARGUMENTS`, `SYNTH_CSA_MAP`, `SYNTH_FA_MAP`, `SYNTH_LATCH_MAP`, `SYNTH_MUX4_MAP`, `SYNTH_MUX_MAP`, `SYNTH_PARAMETERS`, `SYNTH_RCA_MAP`, `SYNTH_TRISTATE_MAP`, `USE_SLANG`, `VERILOG_DEFINES`, `VERILOG_FILES`, `VERILOG_INCLUDE_DIRS`, `VERILOG_POWER_DEFINE`, `YOSYS_LOG_LEVEL`

##### Flow Python Integration
```python
# Add this step to your flow's sequential Steps list:
class CustomRun(SequentialFlow):
    Steps = [
        ...  # prior steps
        Yosys.EQY,
        ...  # subsequent steps
    ]
```
##### Configuration Example
```python
# Configure behavior in the flow's config dict:
config = {
    "SYNTH_LATCH_MAP": None,  # (Optional[librelane.common.types.Path]) A path to a file containing the latch mapping for Yosys.
    "SYNTH_TRISTATE_MAP": None,  # (Optional[librelane.common.types.Path]) A path to a file containing the tri-state buffer mapping for Yosys.
    "SYNTH_CSA_MAP": None,  # (Optional[librelane.common.types.Path]) A path to a file containing the carry-select adder mapping for Yosys.
    "SYNTH_RCA_MAP": None,  # (Optional[librelane.common.types.Path]) A path to a file containing the ripple-carry adder mapping for Yosys.
    "SYNTH_FA_MAP": None,  # (Optional[librelane.common.types.Path]) A path to a file containing the full adder mapping for Yosys.
    "SYNTH_MUX_MAP": None,  # (Optional[librelane.common.types.Path]) A path to a file containing the mux mapping for Yosys.
    "SYNTH_MUX4_MAP": None,  # (Optional[librelane.common.types.Path]) A path to a file containing the mux4 mapping for Yosys.
    "YOSYS_LOG_LEVEL": 'ALL',  # (Literal['ALL', 'WARNING', 'ERROR']) Which log level for Yosys. At WARNING or higher, the initialization splash is...
    "VERILOG_FILES": None,  # (List[librelane.common.types.Path]) The paths of the design's Verilog files.
    "VERILOG_DEFINES": None,  # (Optional[List[str]]) Preprocessor defines for input Verilog files.
    "VERILOG_POWER_DEFINE": 'USE_POWER_PINS',  # (Optional[str]) Specifies the name of the define used to guard power and ground connections i...
    "VERILOG_INCLUDE_DIRS": None,  # (Optional[List[librelane.common.types.Path]]) Specifies the Verilog `include` directories.
    "SYNTH_PARAMETERS": None,  # (Optional[List[str]]) Key-value pairs to be `chparam`ed in Yosys, in the format `key1=value1`.
    "USE_SLANG": False,  # (bool) Use the Slang frontend to process files, which has better SystemVerilog parsi...
    "SLANG_ARGUMENTS": None,  # (Optional[List[str]]) Pass arguments to the Slang frontend.
    "EQY_SCRIPT": None,  # (Optional[librelane.common.types.Path]) The EQY script to use. If unset, a generic EQY script will be generated, but ...
    "MACRO_PLACEMENT_CFG": None,  # (Optional[librelane.common.types.Path]) This step will warn if this deprecated variable is used, as it indicates Macr...
    "EQY_FORCE_ACCEPT_PDK": False,  # (bool) Attempt to run EQY even if the PDK's Verilog models are supported by this ste...
}
```

#### `Yosys.JsonHeader`
Extracts a high-level hierarchical view of the circuit in JSON format,
including power connections. The power connections are used in later steps
to ensure macros and cells are connected as desired.

* **Inputs**: None
* **Outputs**: `json_h` (Design JSON Header File, `.h.json`)
* **Precursor Steps (Dependencies)**:
None (typically runs at the start of the flow, or only requires static design files)
* **Relevant Step-Specific Config**: `SLANG_ARGUMENTS`, `SYNTH_CLOCKGATE_MIN_WIDTH`, `SYNTH_CLOCKGATE_NEGEDGE_ICG`, `SYNTH_CLOCKGATE_POSEDGE_ICG`, `SYNTH_CORNER`, `SYNTH_CSA_MAP`, `SYNTH_FA_MAP`, `SYNTH_LATCH_MAP`, `SYNTH_MUX4_MAP`, `SYNTH_MUX_MAP`, `SYNTH_PARAMETERS`, `SYNTH_RCA_MAP`, `SYNTH_SHOW`, `SYNTH_TRISTATE_MAP`, `USE_SLANG`, `VERILOG_DEFINES`, `VERILOG_FILES`, `VERILOG_INCLUDE_DIRS`, `VERILOG_POWER_DEFINE`, `YOSYS_LOG_LEVEL`

##### Flow Python Integration
```python
# Add this step to your flow's sequential Steps list:
class CustomRun(SequentialFlow):
    Steps = [
        ...  # prior steps
        Yosys.JsonHeader,
        ...  # subsequent steps
    ]
```
##### Configuration Example
```python
# Configure behavior in the flow's config dict:
config = {
    "SYNTH_LATCH_MAP": None,  # (Optional[librelane.common.types.Path]) A path to a file containing the latch mapping for Yosys.
    "SYNTH_TRISTATE_MAP": None,  # (Optional[librelane.common.types.Path]) A path to a file containing the tri-state buffer mapping for Yosys.
    "SYNTH_CSA_MAP": None,  # (Optional[librelane.common.types.Path]) A path to a file containing the carry-select adder mapping for Yosys.
    "SYNTH_RCA_MAP": None,  # (Optional[librelane.common.types.Path]) A path to a file containing the ripple-carry adder mapping for Yosys.
    "SYNTH_FA_MAP": None,  # (Optional[librelane.common.types.Path]) A path to a file containing the full adder mapping for Yosys.
    "SYNTH_MUX_MAP": None,  # (Optional[librelane.common.types.Path]) A path to a file containing the mux mapping for Yosys.
    "SYNTH_MUX4_MAP": None,  # (Optional[librelane.common.types.Path]) A path to a file containing the mux4 mapping for Yosys.
    "SYNTH_CLOCKGATE_MIN_WIDTH": None,  # (Optional[int]) If set to a value, a group of flip-flops with size >= SYNTH_CLOCKGATE_MIN_WID...
    "SYNTH_CLOCKGATE_POSEDGE_ICG": None,  # (Optional[str]) The integrated clock gate cell used for positive-edge flip-flops, in the form...
    "SYNTH_CLOCKGATE_NEGEDGE_ICG": None,  # (Optional[str]) The integrated clock gate cell used for positive-edge flip-flops, in the form...
    "YOSYS_LOG_LEVEL": 'ALL',  # (Literal['ALL', 'WARNING', 'ERROR']) Which log level for Yosys. At WARNING or higher, the initialization splash is...
    "SYNTH_CORNER": None,  # (Optional[str]) A fully qualified IPVT corner to use during synthesis. If unspecified, the va...
    "SYNTH_SHOW": False,  # (bool) Generate a graphviz DOT file for the design. This will fail on a completely e...
    "VERILOG_FILES": None,  # (List[librelane.common.types.Path]) The paths of the design's Verilog files.
    "VERILOG_DEFINES": None,  # (Optional[List[str]]) Preprocessor defines for input Verilog files.
    "VERILOG_POWER_DEFINE": 'USE_POWER_PINS',  # (Optional[str]) Specifies the name of the define used to guard power and ground connections i...
    "VERILOG_INCLUDE_DIRS": None,  # (Optional[List[librelane.common.types.Path]]) Specifies the Verilog `include` directories.
    "SYNTH_PARAMETERS": None,  # (Optional[List[str]]) Key-value pairs to be `chparam`ed in Yosys, in the format `key1=value1`.
    "USE_SLANG": False,  # (bool) Use the Slang frontend to process files, which has better SystemVerilog parsi...
    "SLANG_ARGUMENTS": None,  # (Optional[List[str]]) Pass arguments to the Slang frontend.
}
```

#### `Yosys.Resynthesis`
Like ``Synthesis``, but operates on the input netlist instead of RTL files.
Useful to process/elaborate on netlists generated by tools other than Yosys.

Some metrics will also be extracted and updated, namely:

* ``design__instance__count``
* ``design__instance_unmapped__count``
* ``design__instance__area``

Note that Yosys steps do not currently support gzipped standard cell dotlib
files. They are however supported for macros:

https://github.com/YosysHQ/yosys/issues/4830

* **Inputs**: `nl` (Verilog Netlist, `.nl.v`)
* **Outputs**: `nl` (Verilog Netlist, `.nl.v`)
* **Precursor Steps (Dependencies)**:
* Requires `nl` (Verilog Netlist, `.nl.v`), produced by: `Yosys.Synthesis`, `OpenROAD.Floorplan`, `Odb.DiodesOnPorts` (or other steps in Odb, OpenROAD, Yosys)
* **Relevant Step-Specific Config**: `SYNTH_ABC_AREA_USE_NF`, `SYNTH_ABC_BUFFERING`, `SYNTH_ABC_DFF`, `SYNTH_ABC_LEGACY_REFACTOR`, `SYNTH_ABC_LEGACY_REWRITE`, `SYNTH_ABC_USE_MFS3`, `SYNTH_ADDER_TYPE`, `SYNTH_AUTONAME`, `SYNTH_CHECKS_ALLOW_TRISTATE`, `SYNTH_CLOCKGATE_MIN_WIDTH`, `SYNTH_CLOCKGATE_NEGEDGE_ICG`, `SYNTH_CLOCKGATE_POSEDGE_ICG`, `SYNTH_CORNER`, `SYNTH_CSA_MAP`, `SYNTH_DIRECT_WIRE_BUFFERING`, `SYNTH_ELABORATE_ONLY`, `SYNTH_EXTRA_MAPPING_FILE`, `SYNTH_FA_MAP`, `SYNTH_HIERARCHY_MODE`, `SYNTH_KEEP_HIERARCHY_INSTANCES`, `SYNTH_KEEP_HIERARCHY_MIN_COST`, `SYNTH_KEEP_HIERARCHY_MODULES`, `SYNTH_LATCH_MAP`, `SYNTH_MUL_BOOTH`, `SYNTH_MUX4_MAP`, `SYNTH_MUX_MAP`, `SYNTH_NORMALIZE_SINGLE_BIT_VECTORS`, `SYNTH_RCA_MAP`, `SYNTH_SHARE_RESOURCES`, `SYNTH_SHOW`, `SYNTH_SIZING`, `SYNTH_SPLITNETS`, `SYNTH_STRATEGY`, `SYNTH_TIE_UNDEFINED`, `SYNTH_TRISTATE_MAP`, `SYNTH_WRITE_NOATTR`, `YOSYS_LOG_LEVEL`

##### Flow Python Integration
```python
# Add this step to your flow's sequential Steps list:
class CustomRun(SequentialFlow):
    Steps = [
        ...  # prior steps
        Yosys.Resynthesis,
        ...  # subsequent steps
    ]
```
##### Configuration Example
```python
# Configure behavior in the flow's config dict:
config = {
    "SYNTH_LATCH_MAP": None,  # (Optional[librelane.common.types.Path]) A path to a file containing the latch mapping for Yosys.
    "SYNTH_TRISTATE_MAP": None,  # (Optional[librelane.common.types.Path]) A path to a file containing the tri-state buffer mapping for Yosys.
    "SYNTH_CSA_MAP": None,  # (Optional[librelane.common.types.Path]) A path to a file containing the carry-select adder mapping for Yosys.
    "SYNTH_RCA_MAP": None,  # (Optional[librelane.common.types.Path]) A path to a file containing the ripple-carry adder mapping for Yosys.
    "SYNTH_FA_MAP": None,  # (Optional[librelane.common.types.Path]) A path to a file containing the full adder mapping for Yosys.
    "SYNTH_MUX_MAP": None,  # (Optional[librelane.common.types.Path]) A path to a file containing the mux mapping for Yosys.
    "SYNTH_MUX4_MAP": None,  # (Optional[librelane.common.types.Path]) A path to a file containing the mux4 mapping for Yosys.
    "SYNTH_CLOCKGATE_MIN_WIDTH": None,  # (Optional[int]) If set to a value, a group of flip-flops with size >= SYNTH_CLOCKGATE_MIN_WID...
    "SYNTH_CLOCKGATE_POSEDGE_ICG": None,  # (Optional[str]) The integrated clock gate cell used for positive-edge flip-flops, in the form...
    "SYNTH_CLOCKGATE_NEGEDGE_ICG": None,  # (Optional[str]) The integrated clock gate cell used for positive-edge flip-flops, in the form...
    "YOSYS_LOG_LEVEL": 'ALL',  # (Literal['ALL', 'WARNING', 'ERROR']) Which log level for Yosys. At WARNING or higher, the initialization splash is...
    "SYNTH_CORNER": None,  # (Optional[str]) A fully qualified IPVT corner to use during synthesis. If unspecified, the va...
    "SYNTH_SHOW": False,  # (bool) Generate a graphviz DOT file for the design. This will fail on a completely e...
    "SYNTH_CHECKS_ALLOW_TRISTATE": True,  # (bool) Ignore multiple-driver warnings if they are connected to tri-state buffers on...
    "SYNTH_AUTONAME": False,  # (bool) Generates names for netlist instances. This results in instance names that ca...
    "SYNTH_STRATEGY": 'AREA 0',  # (Literal['AREA 0', 'AREA 1', 'AREA 2', 'AREA 3', 'DELAY 0', 'DELAY 1', 'DELAY 2', 'DELAY 3', 'DELAY 4']) Strategies for abc logic synthesis and technology mapping. AREA strategies us...
    "SYNTH_ABC_BUFFERING": False,  # (bool) Enables `abc` cell buffering.
    "SYNTH_ABC_LEGACY_REFACTOR": False,  # (bool) Replaces the ABC command `drf -l` with `refactor` which matches older version...
    "SYNTH_ABC_LEGACY_REWRITE": False,  # (bool) Replaces the ABC command `drw -l` with `rewrite` which matches older versions...
    "SYNTH_ABC_DFF": False,  # (bool) Passes D-flipflop cells through ABC for optimization (which can for example, ...
    "SYNTH_ABC_USE_MFS3": False,  # (bool) Experimental: attempts a SAT-based remapping in all area and delay strategies...
    "SYNTH_ABC_AREA_USE_NF": False,  # (bool) Experimental: uses the &nf delay-based mapper with a very high value instead ...
    "SYNTH_DIRECT_WIRE_BUFFERING": True,  # (bool) Enables inserting buffer cells for directly connected wires.
    "SYNTH_SPLITNETS": True,  # (bool) Splits multi-bit nets into single-bit nets. Easier to trace but may not be su...
    "SYNTH_SIZING": False,  # (bool) Enables `abc` cell sizing (instead of buffering).
    "SYNTH_HIERARCHY_MODE": 'flatten',  # (Literal['flatten', 'deferred_flatten', 'keep']) Affects how hierarchy is maintained throughout and after synthesis. 'flatten'...
    "SYNTH_KEEP_HIERARCHY_MIN_COST": None,  # (Optional[int]) Sets the 'keep_hierarchy' attribute on modules where the gate count is estima...
    "SYNTH_KEEP_HIERARCHY_INSTANCES": None,  # (Optional[List[str]]) A list of instances for which to set the 'keep_hierarchy' attribute. This var...
    "SYNTH_KEEP_HIERARCHY_MODULES": None,  # (Optional[List[str]]) A list of modules for which to set the 'keep_hierarchy' attribute. This varia...
    "SYNTH_SHARE_RESOURCES": True,  # (bool) A flag that enables yosys to reduce the number of cells by determining sharea...
    "SYNTH_ADDER_TYPE": 'YOSYS',  # (Literal['YOSYS', 'FA', 'RCA', 'CSA']) Adder type to which the \$add and \$sub operators are mapped to.  Possible valu...
    "SYNTH_EXTRA_MAPPING_FILE": None,  # (Optional[librelane.common.types.Path]) Points to an extra techmap file for yosys that runs right after yosys `synth`...
    "SYNTH_ELABORATE_ONLY": False,  # (bool) "Elaborate" the design only without attempting any logic mapping. Useful when...
    "SYNTH_MUL_BOOTH": False,  # (bool) Runs the booth pass as part of synthesis: See https://yosyshq.readthedocs.io/...
    "SYNTH_TIE_UNDEFINED": 'low',  # (Optional[Literal['high', 'low']]) Whether to tie undefined values low or high. Explicitly provide null if you w...
    "SYNTH_WRITE_NOATTR": True,  # (bool) If true, Verilog-2001 attributes are omitted from output netlists. Some utili...
    "SYNTH_NORMALIZE_SINGLE_BIT_VECTORS": True,  # (bool) If true, vectors with the shape [0:0] are converted to normal wires in the ne...
}
```

#### `Yosys.Synthesis`
Performs synthesis and technology mapping on Verilog RTL files
using Yosys and ABC, emitting a netlist.

Some metrics will also be extracted and updated, namely:

* ``design__instance__count``
* ``design__instance_unmapped__count``
* ``design__instance__area``

Note that Yosys steps do not currently support gzipped standard cell dotlib
files. They are however supported for macros:

https://github.com/YosysHQ/yosys/issues/4830

* **Inputs**: None
* **Outputs**: `nl` (Verilog Netlist, `.nl.v`)
* **Precursor Steps (Dependencies)**:
None (typically runs at the start of the flow, or only requires static design files)
* **Relevant Step-Specific Config**: `SLANG_ARGUMENTS`, `SYNTH_ABC_AREA_USE_NF`, `SYNTH_ABC_BUFFERING`, `SYNTH_ABC_DFF`, `SYNTH_ABC_LEGACY_REFACTOR`, `SYNTH_ABC_LEGACY_REWRITE`, `SYNTH_ABC_USE_MFS3`, `SYNTH_ADDER_TYPE`, `SYNTH_AUTONAME`, `SYNTH_CHECKS_ALLOW_TRISTATE`, `SYNTH_CLOCKGATE_MIN_WIDTH`, `SYNTH_CLOCKGATE_NEGEDGE_ICG`, `SYNTH_CLOCKGATE_POSEDGE_ICG`, `SYNTH_CORNER`, `SYNTH_CSA_MAP`, `SYNTH_DIRECT_WIRE_BUFFERING`, `SYNTH_ELABORATE_ONLY`, `SYNTH_EXTRA_MAPPING_FILE`, `SYNTH_FA_MAP`, `SYNTH_HIERARCHY_MODE`, `SYNTH_KEEP_HIERARCHY_INSTANCES`, `SYNTH_KEEP_HIERARCHY_MIN_COST`, `SYNTH_KEEP_HIERARCHY_MODULES`, `SYNTH_LATCH_MAP`, `SYNTH_MUL_BOOTH`, `SYNTH_MUX4_MAP`, `SYNTH_MUX_MAP`, `SYNTH_NORMALIZE_SINGLE_BIT_VECTORS`, `SYNTH_PARAMETERS`, `SYNTH_RCA_MAP`, `SYNTH_SHARE_RESOURCES`, `SYNTH_SHOW`, `SYNTH_SIZING`, `SYNTH_SPLITNETS`, `SYNTH_STRATEGY`, `SYNTH_TIE_UNDEFINED`, `SYNTH_TRISTATE_MAP`, `SYNTH_WRITE_NOATTR`, `USE_SLANG`, `VERILOG_DEFINES`, `VERILOG_FILES`, `VERILOG_INCLUDE_DIRS`, `VERILOG_POWER_DEFINE`, `YOSYS_LOG_LEVEL`

##### Flow Python Integration
```python
# Add this step to your flow's sequential Steps list:
class CustomRun(SequentialFlow):
    Steps = [
        ...  # prior steps
        Yosys.Synthesis,
        ...  # subsequent steps
    ]
```
##### Configuration Example
```python
# Configure behavior in the flow's config dict:
config = {
    "SYNTH_LATCH_MAP": None,  # (Optional[librelane.common.types.Path]) A path to a file containing the latch mapping for Yosys.
    "SYNTH_TRISTATE_MAP": None,  # (Optional[librelane.common.types.Path]) A path to a file containing the tri-state buffer mapping for Yosys.
    "SYNTH_CSA_MAP": None,  # (Optional[librelane.common.types.Path]) A path to a file containing the carry-select adder mapping for Yosys.
    "SYNTH_RCA_MAP": None,  # (Optional[librelane.common.types.Path]) A path to a file containing the ripple-carry adder mapping for Yosys.
    "SYNTH_FA_MAP": None,  # (Optional[librelane.common.types.Path]) A path to a file containing the full adder mapping for Yosys.
    "SYNTH_MUX_MAP": None,  # (Optional[librelane.common.types.Path]) A path to a file containing the mux mapping for Yosys.
    "SYNTH_MUX4_MAP": None,  # (Optional[librelane.common.types.Path]) A path to a file containing the mux4 mapping for Yosys.
    "SYNTH_CLOCKGATE_MIN_WIDTH": None,  # (Optional[int]) If set to a value, a group of flip-flops with size >= SYNTH_CLOCKGATE_MIN_WID...
    "SYNTH_CLOCKGATE_POSEDGE_ICG": None,  # (Optional[str]) The integrated clock gate cell used for positive-edge flip-flops, in the form...
    "SYNTH_CLOCKGATE_NEGEDGE_ICG": None,  # (Optional[str]) The integrated clock gate cell used for positive-edge flip-flops, in the form...
    "YOSYS_LOG_LEVEL": 'ALL',  # (Literal['ALL', 'WARNING', 'ERROR']) Which log level for Yosys. At WARNING or higher, the initialization splash is...
    "SYNTH_CORNER": None,  # (Optional[str]) A fully qualified IPVT corner to use during synthesis. If unspecified, the va...
    "SYNTH_SHOW": False,  # (bool) Generate a graphviz DOT file for the design. This will fail on a completely e...
    "SYNTH_CHECKS_ALLOW_TRISTATE": True,  # (bool) Ignore multiple-driver warnings if they are connected to tri-state buffers on...
    "SYNTH_AUTONAME": False,  # (bool) Generates names for netlist instances. This results in instance names that ca...
    "SYNTH_STRATEGY": 'AREA 0',  # (Literal['AREA 0', 'AREA 1', 'AREA 2', 'AREA 3', 'DELAY 0', 'DELAY 1', 'DELAY 2', 'DELAY 3', 'DELAY 4']) Strategies for abc logic synthesis and technology mapping. AREA strategies us...
    "SYNTH_ABC_BUFFERING": False,  # (bool) Enables `abc` cell buffering.
    "SYNTH_ABC_LEGACY_REFACTOR": False,  # (bool) Replaces the ABC command `drf -l` with `refactor` which matches older version...
    "SYNTH_ABC_LEGACY_REWRITE": False,  # (bool) Replaces the ABC command `drw -l` with `rewrite` which matches older versions...
    "SYNTH_ABC_DFF": False,  # (bool) Passes D-flipflop cells through ABC for optimization (which can for example, ...
    "SYNTH_ABC_USE_MFS3": False,  # (bool) Experimental: attempts a SAT-based remapping in all area and delay strategies...
    "SYNTH_ABC_AREA_USE_NF": False,  # (bool) Experimental: uses the &nf delay-based mapper with a very high value instead ...
    "SYNTH_DIRECT_WIRE_BUFFERING": True,  # (bool) Enables inserting buffer cells for directly connected wires.
    "SYNTH_SPLITNETS": True,  # (bool) Splits multi-bit nets into single-bit nets. Easier to trace but may not be su...
    "SYNTH_SIZING": False,  # (bool) Enables `abc` cell sizing (instead of buffering).
    "SYNTH_HIERARCHY_MODE": 'flatten',  # (Literal['flatten', 'deferred_flatten', 'keep']) Affects how hierarchy is maintained throughout and after synthesis. 'flatten'...
    "SYNTH_KEEP_HIERARCHY_MIN_COST": None,  # (Optional[int]) Sets the 'keep_hierarchy' attribute on modules where the gate count is estima...
    "SYNTH_KEEP_HIERARCHY_INSTANCES": None,  # (Optional[List[str]]) A list of instances for which to set the 'keep_hierarchy' attribute. This var...
    "SYNTH_KEEP_HIERARCHY_MODULES": None,  # (Optional[List[str]]) A list of modules for which to set the 'keep_hierarchy' attribute. This varia...
    "SYNTH_SHARE_RESOURCES": True,  # (bool) A flag that enables yosys to reduce the number of cells by determining sharea...
    "SYNTH_ADDER_TYPE": 'YOSYS',  # (Literal['YOSYS', 'FA', 'RCA', 'CSA']) Adder type to which the \$add and \$sub operators are mapped to.  Possible valu...
    "SYNTH_EXTRA_MAPPING_FILE": None,  # (Optional[librelane.common.types.Path]) Points to an extra techmap file for yosys that runs right after yosys `synth`...
    "SYNTH_ELABORATE_ONLY": False,  # (bool) "Elaborate" the design only without attempting any logic mapping. Useful when...
    "SYNTH_MUL_BOOTH": False,  # (bool) Runs the booth pass as part of synthesis: See https://yosyshq.readthedocs.io/...
    "SYNTH_TIE_UNDEFINED": 'low',  # (Optional[Literal['high', 'low']]) Whether to tie undefined values low or high. Explicitly provide null if you w...
    "SYNTH_WRITE_NOATTR": True,  # (bool) If true, Verilog-2001 attributes are omitted from output netlists. Some utili...
    "SYNTH_NORMALIZE_SINGLE_BIT_VECTORS": True,  # (bool) If true, vectors with the shape [0:0] are converted to normal wires in the ne...
    "VERILOG_FILES": None,  # (List[librelane.common.types.Path]) The paths of the design's Verilog files.
    "VERILOG_DEFINES": None,  # (Optional[List[str]]) Preprocessor defines for input Verilog files.
    "VERILOG_POWER_DEFINE": 'USE_POWER_PINS',  # (Optional[str]) Specifies the name of the define used to guard power and ground connections i...
    "VERILOG_INCLUDE_DIRS": None,  # (Optional[List[librelane.common.types.Path]]) Specifies the Verilog `include` directories.
    "SYNTH_PARAMETERS": None,  # (Optional[List[str]]) Key-value pairs to be `chparam`ed in Yosys, in the format `key1=value1`.
    "USE_SLANG": False,  # (bool) Use the Slang frontend to process files, which has better SystemVerilog parsi...
    "SLANG_ARGUMENTS": None,  # (Optional[List[str]]) Pass arguments to the Slang frontend.
}
```

#### `Yosys.VHDLSynthesis`
Performs synthesis and technology mapping on VHDL files
using Yosys, GHDL and ABC, emitting a netlist.

Some metrics will also be extracted and updated, namely:

* ``design__instance__count``
* ``design__instance_unmapped__count``
* ``design__instance__area``

Note that Yosys steps do not currently support gzipped standard cell dotlib
files. They are however supported for macros:

https://github.com/YosysHQ/yosys/issues/4830

* **Inputs**: None
* **Outputs**: `nl` (Verilog Netlist, `.nl.v`)
* **Precursor Steps (Dependencies)**:
None (typically runs at the start of the flow, or only requires static design files)
* **Relevant Step-Specific Config**: `GHDL_ARGUMENTS`, `SYNTH_ABC_AREA_USE_NF`, `SYNTH_ABC_BUFFERING`, `SYNTH_ABC_DFF`, `SYNTH_ABC_LEGACY_REFACTOR`, `SYNTH_ABC_LEGACY_REWRITE`, `SYNTH_ABC_USE_MFS3`, `SYNTH_ADDER_TYPE`, `SYNTH_AUTONAME`, `SYNTH_CHECKS_ALLOW_TRISTATE`, `SYNTH_CLOCKGATE_MIN_WIDTH`, `SYNTH_CLOCKGATE_NEGEDGE_ICG`, `SYNTH_CLOCKGATE_POSEDGE_ICG`, `SYNTH_CORNER`, `SYNTH_CSA_MAP`, `SYNTH_DIRECT_WIRE_BUFFERING`, `SYNTH_ELABORATE_ONLY`, `SYNTH_EXTRA_MAPPING_FILE`, `SYNTH_FA_MAP`, `SYNTH_HIERARCHY_MODE`, `SYNTH_KEEP_HIERARCHY_INSTANCES`, `SYNTH_KEEP_HIERARCHY_MIN_COST`, `SYNTH_KEEP_HIERARCHY_MODULES`, `SYNTH_LATCH_MAP`, `SYNTH_MUL_BOOTH`, `SYNTH_MUX4_MAP`, `SYNTH_MUX_MAP`, `SYNTH_NORMALIZE_SINGLE_BIT_VECTORS`, `SYNTH_RCA_MAP`, `SYNTH_SHARE_RESOURCES`, `SYNTH_SHOW`, `SYNTH_SIZING`, `SYNTH_SPLITNETS`, `SYNTH_STRATEGY`, `SYNTH_TIE_UNDEFINED`, `SYNTH_TRISTATE_MAP`, `SYNTH_WRITE_NOATTR`, `VHDL_FILES`, `YOSYS_LOG_LEVEL`

##### Flow Python Integration
```python
# Add this step to your flow's sequential Steps list:
class CustomRun(SequentialFlow):
    Steps = [
        ...  # prior steps
        Yosys.VHDLSynthesis,
        ...  # subsequent steps
    ]
```
##### Configuration Example
```python
# Configure behavior in the flow's config dict:
config = {
    "SYNTH_LATCH_MAP": None,  # (Optional[librelane.common.types.Path]) A path to a file containing the latch mapping for Yosys.
    "SYNTH_TRISTATE_MAP": None,  # (Optional[librelane.common.types.Path]) A path to a file containing the tri-state buffer mapping for Yosys.
    "SYNTH_CSA_MAP": None,  # (Optional[librelane.common.types.Path]) A path to a file containing the carry-select adder mapping for Yosys.
    "SYNTH_RCA_MAP": None,  # (Optional[librelane.common.types.Path]) A path to a file containing the ripple-carry adder mapping for Yosys.
    "SYNTH_FA_MAP": None,  # (Optional[librelane.common.types.Path]) A path to a file containing the full adder mapping for Yosys.
    "SYNTH_MUX_MAP": None,  # (Optional[librelane.common.types.Path]) A path to a file containing the mux mapping for Yosys.
    "SYNTH_MUX4_MAP": None,  # (Optional[librelane.common.types.Path]) A path to a file containing the mux4 mapping for Yosys.
    "SYNTH_CLOCKGATE_MIN_WIDTH": None,  # (Optional[int]) If set to a value, a group of flip-flops with size >= SYNTH_CLOCKGATE_MIN_WID...
    "SYNTH_CLOCKGATE_POSEDGE_ICG": None,  # (Optional[str]) The integrated clock gate cell used for positive-edge flip-flops, in the form...
    "SYNTH_CLOCKGATE_NEGEDGE_ICG": None,  # (Optional[str]) The integrated clock gate cell used for positive-edge flip-flops, in the form...
    "YOSYS_LOG_LEVEL": 'ALL',  # (Literal['ALL', 'WARNING', 'ERROR']) Which log level for Yosys. At WARNING or higher, the initialization splash is...
    "SYNTH_CORNER": None,  # (Optional[str]) A fully qualified IPVT corner to use during synthesis. If unspecified, the va...
    "SYNTH_SHOW": False,  # (bool) Generate a graphviz DOT file for the design. This will fail on a completely e...
    "SYNTH_CHECKS_ALLOW_TRISTATE": True,  # (bool) Ignore multiple-driver warnings if they are connected to tri-state buffers on...
    "SYNTH_AUTONAME": False,  # (bool) Generates names for netlist instances. This results in instance names that ca...
    "SYNTH_STRATEGY": 'AREA 0',  # (Literal['AREA 0', 'AREA 1', 'AREA 2', 'AREA 3', 'DELAY 0', 'DELAY 1', 'DELAY 2', 'DELAY 3', 'DELAY 4']) Strategies for abc logic synthesis and technology mapping. AREA strategies us...
    "SYNTH_ABC_BUFFERING": False,  # (bool) Enables `abc` cell buffering.
    "SYNTH_ABC_LEGACY_REFACTOR": False,  # (bool) Replaces the ABC command `drf -l` with `refactor` which matches older version...
    "SYNTH_ABC_LEGACY_REWRITE": False,  # (bool) Replaces the ABC command `drw -l` with `rewrite` which matches older versions...
    "SYNTH_ABC_DFF": False,  # (bool) Passes D-flipflop cells through ABC for optimization (which can for example, ...
    "SYNTH_ABC_USE_MFS3": False,  # (bool) Experimental: attempts a SAT-based remapping in all area and delay strategies...
    "SYNTH_ABC_AREA_USE_NF": False,  # (bool) Experimental: uses the &nf delay-based mapper with a very high value instead ...
    "SYNTH_DIRECT_WIRE_BUFFERING": True,  # (bool) Enables inserting buffer cells for directly connected wires.
    "SYNTH_SPLITNETS": True,  # (bool) Splits multi-bit nets into single-bit nets. Easier to trace but may not be su...
    "SYNTH_SIZING": False,  # (bool) Enables `abc` cell sizing (instead of buffering).
    "SYNTH_HIERARCHY_MODE": 'flatten',  # (Literal['flatten', 'deferred_flatten', 'keep']) Affects how hierarchy is maintained throughout and after synthesis. 'flatten'...
    "SYNTH_KEEP_HIERARCHY_MIN_COST": None,  # (Optional[int]) Sets the 'keep_hierarchy' attribute on modules where the gate count is estima...
    "SYNTH_KEEP_HIERARCHY_INSTANCES": None,  # (Optional[List[str]]) A list of instances for which to set the 'keep_hierarchy' attribute. This var...
    "SYNTH_KEEP_HIERARCHY_MODULES": None,  # (Optional[List[str]]) A list of modules for which to set the 'keep_hierarchy' attribute. This varia...
    "SYNTH_SHARE_RESOURCES": True,  # (bool) A flag that enables yosys to reduce the number of cells by determining sharea...
    "SYNTH_ADDER_TYPE": 'YOSYS',  # (Literal['YOSYS', 'FA', 'RCA', 'CSA']) Adder type to which the \$add and \$sub operators are mapped to.  Possible valu...
    "SYNTH_EXTRA_MAPPING_FILE": None,  # (Optional[librelane.common.types.Path]) Points to an extra techmap file for yosys that runs right after yosys `synth`...
    "SYNTH_ELABORATE_ONLY": False,  # (bool) "Elaborate" the design only without attempting any logic mapping. Useful when...
    "SYNTH_MUL_BOOTH": False,  # (bool) Runs the booth pass as part of synthesis: See https://yosyshq.readthedocs.io/...
    "SYNTH_TIE_UNDEFINED": 'low',  # (Optional[Literal['high', 'low']]) Whether to tie undefined values low or high. Explicitly provide null if you w...
    "SYNTH_WRITE_NOATTR": True,  # (bool) If true, Verilog-2001 attributes are omitted from output netlists. Some utili...
    "SYNTH_NORMALIZE_SINGLE_BIT_VECTORS": True,  # (bool) If true, vectors with the shape [0:0] are converted to normal wires in the ne...
    "VHDL_FILES": None,  # (List[librelane.common.types.Path]) The paths of the design's VHDL files.
    "GHDL_ARGUMENTS": None,  # (Optional[List[str]]) Pass arguments to the ghdl frontend.
}
```

## 4. Configuration Variables Reference
This section contains complete reference tables of all available configuration variables in LibreLane.

### Universal Flow Variables
These variables apply globally to the flow execution.


| Variable Name                         | Type                                                         | Description                                                                                                                                                                                                                                                                      | Default                                                                                       | Units |
| ------------------------------------- | ------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------- | ----- |
| `CELL_BB_VERILOG_MODELS`  [PDK]       | List[Path]?                                                  | Path(s) to cells' black-box Verilog model(s)                                                                                                                                                                                                                                     | `None`                                                                                        |       |
| `CELL_CDLS`  [PDK]                    | List[Path]?                                                  | A circuit-design language view of the standard cell library.                                                                                                                                                                                                                     | `None`                                                                                        |       |
| `CELL_GDS`  [PDK]                     | List[Path]                                                   | Path(s) to the cells' GDSII file(s).                                                                                                                                                                                                                                             | `None`                                                                                        |       |
| `CELL_LEFS`  [PDK]                    | List[Path]                                                   | Path(s) to the cells' LEF file(s).                                                                                                                                                                                                                                               | `None`                                                                                        |       |
| `CELL_PAD_EXCLUDE`  [PDK]             | List[str]                                                    | Defines a list of cells to be excluded from cell padding.                                                                                                                                                                                                                        | `None`                                                                                        |       |
| `CELL_SPICE_MODELS`  [PDK]            | List[Path]?                                                  | Path(s) to cells' SPICE model(s)                                                                                                                                                                                                                                                 | `None`                                                                                        |       |
| `CELL_VERILOG_MODELS`  [PDK]          | List[Path]?                                                  | Path(s) to cells' Verilog model(s)                                                                                                                                                                                                                                               | `None`                                                                                        |       |
| `CLOCK_NET`                           | (str｜ List[str])?                                            | The name of the net input to root clock buffer. If unset, it is presumed to be equal to CLOCK_PORT.                                                                                                                                                                              | `None`                                                                                        |       |
| `CLOCK_PERIOD`                        | Decimal                                                      | The clock period for the design.                                                                                                                                                                                                                                                 | `10.0`                                                                                        | ns    |
| `CLOCK_PORT`                          | (str｜ List[str])?                                            | The name(s) of the design's clock port(s).                                                                                                                                                                                                                                       | `None`                                                                                        |       |
| `CLOCK_TRANSITION_CONSTRAINT`  [PDK]  | Decimal                                                      | Specifies a value for the clock transition/slew for timing analysis.                                                                                                                                                                                                             | `None`                                                                                        | ns    |
| `CLOCK_UNCERTAINTY_CONSTRAINT`  [PDK] | Decimal                                                      | Specifies a value for the clock uncertainty/jitter for timing analysis.                                                                                                                                                                                                          | `None`                                                                                        | ns    |
| `DECAP_CELLS`  [PDK]                  | List[str]                                                    | A list of cell names or wildcards of decap cells to be used in fill insertion.                                                                                                                                                                                                   | `None`                                                                                        |       |
| `DEFAULT_CORNER`  [PDK]               | str                                                          | The interconnect/process/voltage/temperature corner (IPVT) to use the characterized lib files compatible with by default.                                                                                                                                                        | `None`                                                                                        |       |
| `DEFAULT_MAX_TRAN`  [PDK]             | Decimal?                                                     | Defines the default maximum transition value used in Synthesis and CTS. A minimum of 0.1 * CLOCK_PERIOD and this variable, if defined, is used.                                                                                                                                  | `None`                                                                                        | ns    |
| `DESIGN_DIR`                          | Path                                                         | The directory of the design. Should be set via command-line arguments or :meth:`Config.load` flags and not actual configuration files. If using a configuration file, ``DESIGN_DIR`` will be the directory where that file exists.                                               | `None`                                                                                        |       |
| `DESIGN_NAME`                         | str                                                          | The name of the top level module of the design. Must be a valid C identifier, i.e., matches the regular expression `[_a-zA-Z][_a-zA-Z0-9]+`.                                                                                                                                     | `None`                                                                                        |       |
| `DIE_AREA`                            | Tuple[Decimal, Decimal, Decimal, Decimal]?                   | Specific die area to be used in floorplanning. Specified as a 4-corner rectangle "x0 y0 x1 y1".                                                                                                                                                                                  | `None`                                                                                        | µm    |
| `DIODE_CELL`  [PDK]                   | str?                                                         | Defines a diode cell used to fix antenna violations, in the format `{cell}/{port}`. If not defined, steps should not attempt to repair the antenna effect by inserting diode cells.                                                                                              | `None`                                                                                        |       |
| `ENDCAP_CELL`  [PDK]                  | str?                                                         | Defines the so-called 'end-cap' cell- class of decap cells placed at either sides of a design, if available.                                                                                                                                                                     | `None`                                                                                        |       |
| `EXTRA_CDLS`                          | List[Path]?                                                  | Specifies miscellaneous CDL netlists to be loaded indiscriminately whenever CDL netlists are loaded.                                                                                                                                                                             | `None`                                                                                        |       |
| `EXTRA_EXCLUDED_CELLS`                | List[str]?                                                   | Wildcards matching additional cells to exclude from both synthesis and PnR.                                                                                                                                                                                                      | `None`                                                                                        |       |
| `EXTRA_GDS`                           | List[Path]?                                                  | Specifies GDS files of pre-hardened macros used in the current design, used during tape-out.                                                                                                                                                                                     | `None`                                                                                        |       |
| `EXTRA_LEFS`                          | List[Path]?                                                  | Specifies miscellaneous LEF files to be loaded indiscriminately whenever LEFs are loaded.                                                                                                                                                                                        | `None`                                                                                        |       |
| `EXTRA_LIBS`                          | List[Path]?                                                  | Specifies LIB files of pre-hardened macros used in the current design, used during timing analyses (and during parasitics-based STA as a fallback). These are loaded indiscriminately for all timing corners.                                                                    | `None`                                                                                        |       |
| `EXTRA_SPICE_MODELS`                  | List[Path]?                                                  | Specifies miscellaneous SPICE models to be loaded indiscriminately whenever SPICE models are loaded.                                                                                                                                                                             | `None`                                                                                        |       |
| `EXTRA_VERILOG_MODELS`                | List[Path]?                                                  | Specifies miscellaneous Verilog models to be loaded indiscriminately during synthesis.                                                                                                                                                                                           | `None`                                                                                        |       |
| `FALLBACK_SDC`                        | Path                                                         | A fallback SDC file for when a step-specific SDC file is not defined.                                                                                                                                                                                                            | `/home/vscode/Desktop/LibreLane/venv/lib/python3.10/site-packages/librelane/scripts/base.sdc` |       |
| `FILL_CELLS`  [PDK]                   | List[str]                                                    | A list of cell names or wildcards of fill cells to be used in fill insertion.                                                                                                                                                                                                    | `None`                                                                                        |       |
| `GND_NETS`                            | List[str]?                                                   | Specifies the ground nets/pins to be used when creating the power grid for the design.                                                                                                                                                                                           | `None`                                                                                        |       |
| `GND_PIN`  [PDK]                      | str                                                          | The ground pin for the cells.                                                                                                                                                                                                                                                    | `None`                                                                                        |       |
| `IO_DELAY_CONSTRAINT`  [PDK]          | Decimal                                                      | Specifies the percentage of the clock period used in the input/output delays.                                                                                                                                                                                                    | `None`                                                                                        | %     |
| `LIB`  [PDK]                          | Dict[str, List[Path]]                                        | A map from corner patterns to a list of associated liberty files. Exactly one entry must match the `DEFAULT_CORNER`.                                                                                                                                                             | `None`                                                                                        |       |
| `MACROS`                              | Dict[str, {class}`Macro <librelane.config.variable.Macro>`]? | A dictionary of Macro definition objects. See {py:class}`librelane.config.Macro` for more info.                                                                                                                                                                                  | `None`                                                                                        |       |
| `MAX_CAPACITANCE_CONSTRAINT`  [PDK]   | Decimal?                                                     | The maximum capacitance constraint. If not provided, the constraint is not set in the SDC file which will fall back to the value set by the liberty file                                                                                                                         | `None`                                                                                        | pF    |
| `MAX_FANOUT_CONSTRAINT`  [PDK]        | int                                                          | The max load that the output ports can drive to be used as a constraint on Synthesis and CTS.                                                                                                                                                                                    | `None`                                                                                        | cells |
| `MAX_TRANSITION_CONSTRAINT`  [PDK]    | Decimal?                                                     | The max transition time (slew) from high to low or low to high on cell inputs in ns to be used as a constraint on Synthesis and CTS. If not provided, it is calculated at runtime as `10%` of the provided clock period, unless that exceeds the PDK's `DEFAULT_MAX_TRAN` value. | `None`                                                                                        | ns    |
| `OUTPUT_CAP_LOAD`  [PDK]              | Decimal                                                      | Defines the capacitive load on the output ports.                                                                                                                                                                                                                                 | `None`                                                                                        | fF    |
| `PAD_BONDPAD_HEIGHT`  [PDK]           | Decimal?                                                     | Height of the bondpad.                                                                                                                                                                                                                                                           | `None`                                                                                        | µm    |
| `PAD_BONDPAD_NAME`  [PDK]             | str?                                                         | Name of the bondpad cell, if empty, bondpads won't be placed.                                                                                                                                                                                                                    | `None`                                                                                        |       |
| `PAD_BONDPAD_OFFSETS`  [PDK]          | Dict[str, Tuple[Decimal, Decimal]]?                          | A dict of pad master names or regular expressions to their bondpad (offset_x, offset_y) tuple.                                                                                                                                                                                   | `None`                                                                                        |       |
| `PAD_BONDPAD_WIDTH`  [PDK]            | Decimal?                                                     | Width of the bondpad.                                                                                                                                                                                                                                                            | `None`                                                                                        | µm    |
| `PAD_CDLS`  [PDK]                     | List[Path]?                                                  | A circuit-design language view of the io pad library.                                                                                                                                                                                                                            | `None`                                                                                        |       |
| `PAD_CORNER`  [PDK]                   | List[str]?                                                   | The pad corner cell.                                                                                                                                                                                                                                                             | `None`                                                                                        |       |
| `PAD_CORNER_SITE_NAME`  [PDK]         | str?                                                         | Name of the corner site.                                                                                                                                                                                                                                                         | `None`                                                                                        | µm    |
| `PAD_EDGE_SPACING`  [PDK]             | Decimal?                                                     | Distance from the padring to the die boundary. Used to account for the sealring when placing the pads.                                                                                                                                                                           | `0`                                                                                           | µm    |
| `PAD_FAKE_SITES`  [PDK]               | Dict[str, Tuple[Decimal, Decimal]]?                          | A dict of fake pad sites and their width and height tuple. Use this if the LEF does not include the site definitions for the IO pads.                                                                                                                                            | `None`                                                                                        | µm    |
| `PAD_FILLERS`  [PDK]                  | List[str]?                                                   | A list of pad filler cells.                                                                                                                                                                                                                                                      | `None`                                                                                        |       |
| `PAD_GDS`  [PDK]                      | List[Path]?                                                  | Path(s) to IO pad GDS file(s).                                                                                                                                                                                                                                                   | `None`                                                                                        |       |
| `PAD_LEFS`  [PDK]                     | List[Path]?                                                  | Path(s) to IO pad LEF file(s).                                                                                                                                                                                                                                                   | `None`                                                                                        |       |
| `PAD_LIBS`  [PDK]                     | Dict[str, List[Path]]?                                       | A map from corner patterns to a list of associated liberty files. Exactly one entry must match the `DEFAULT_CORNER`.                                                                                                                                                             | `None`                                                                                        |       |
| `PAD_PLACE_IO_TERMINALS`  [PDK]       | List[str]?                                                   | Place I/O terminals for these master/pin combinations.                                                                                                                                                                                                                           | `None`                                                                                        |       |
| `PAD_SITE_NAME`  [PDK]                | str?                                                         | Name of the pad site.                                                                                                                                                                                                                                                            | `None`                                                                                        | µm    |
| `PAD_SPICE_MODELS`  [PDK]             | List[Path]?                                                  | Path(s) to IO pads' SPICE model(s)                                                                                                                                                                                                                                               | `None`                                                                                        |       |
| `PAD_VERILOG_MODELS`  [PDK]           | List[Path]?                                                  | Path(s) to IO pads' Verilog model(s)                                                                                                                                                                                                                                             | `None`                                                                                        |       |
| `PDK`                                 | str                                                          | Specifies the process design kit (PDK). Must be a valid C identifier, i.e., matches the regular expression `[_a-zA-Z][_a-zA-Z0-9]+`.                                                                                                                                             | `sky130A`                                                                                     |       |
| `PDK_ROOT`                            | Path                                                         | The home path of all PDKs. Should be set via command-line arguments or :meth:`Config.load` flags and not actual configuration files.                                                                                                                                             | `None`                                                                                        |       |
| `PLACE_SITE`  [PDK]                   | str                                                          | Defines the primary placement site in placement as specified in the technology LEF files, to generate the placement grid.                                                                                                                                                        | `None`                                                                                        |       |
| `PNR_EXCLUDED_CELL_FILE`  [PDK]       | Path                                                         | Path to a text file containing a list of undesirable or bad (DRC-failed or complex pinout) cells or wildcards matching cells to be excluded from synthesis AND PnR.                                                                                                              | `None`                                                                                        |       |
| `PRIMARY_GDSII_STREAMOUT_TOOL`  [PDK] | str                                                          | Specify the primary GDSII streamout tool for this PDK. For most open-source PDKs, that would be 'magic'.                                                                                                                                                                         | `None`                                                                                        |       |
| `RT_MAX_LAYER`  [PDK]                 | str                                                          | The highest metal layer to route on.                                                                                                                                                                                                                                             | `None`                                                                                        |       |
| `RT_MIN_LAYER`  [PDK]                 | str                                                          | The lowest metal layer to route on.                                                                                                                                                                                                                                              | `None`                                                                                        |       |
| `SCL_GROUND_PINS`  [PDK]              | List[str]                                                    | SCL-specific ground pins                                                                                                                                                                                                                                                         | `None`                                                                                        |       |
| `SCL_POWER_PINS`  [PDK]               | List[str]                                                    | SCL-specific power pins                                                                                                                                                                                                                                                          | `None`                                                                                        |       |
| `STA_CORNERS`  [PDK]                  | List[str]                                                    | A list of fully qualified IPVT (Interconnect, transistor Process, Voltage, and Temperature) timing corners on which to conduct multi-corner static timing analysis.                                                                                                              | `None`                                                                                        |       |
| `STD_CELL_LIBRARY`  [PDK]             | str                                                          | Specifies the default standard cell library to be used under the specified PDK. Must be a valid C identifier, i.e., matches the regular expression `[_a-zA-Z][_a-zA-Z0-9]+`.                                                                                                     | `None`                                                                                        |       |
| `SYNTH_BUFFER_CELL`  [PDK]            | str                                                          | Defines a buffer port to be used by yosys during synthesis: in the format `{cell}/{input_port}/{output_port}`                                                                                                                                                                    | `None`                                                                                        |       |
| `SYNTH_CLK_DRIVING_CELL`  [PDK]       | str?                                                         | The cell to drive the clock input ports, used in synthesis and static timing analysis, in the format `{cell}/{port}`. If not specified, `SYNTH_DRIVING_CELL` will be used.                                                                                                       | `None`                                                                                        |       |
| `SYNTH_DRIVING_CELL`  [PDK]           | str                                                          | The cell to drive the input ports, used in synthesis and static timing analysis, in the format `{cell}/{port}`.                                                                                                                                                                  | `None`                                                                                        |       |
| `SYNTH_EXCLUDED_CELL_FILE`  [PDK]     | Path                                                         | Path to a text file containing a list of (wildcards matching) cells to be excluded from the lib file in synthesis alone.                                                                                                                                                         | `None`                                                                                        |       |
| `SYNTH_TIEHI_CELL`  [PDK]             | str                                                          | Defines the tie high cell followed by the port that implements the tie high functionality, in the format `{cell}/{port}`.                                                                                                                                                        | `None`                                                                                        |       |
| `SYNTH_TIELO_CELL`  [PDK]             | str                                                          | Defines the tie high cell followed by the port that implements the tie low functionality, in the format `{cell}/{port}`.                                                                                                                                                         | `None`                                                                                        |       |
| `TECH_LEFS`  [PDK]                    | Dict[str, Path]                                              | Map of corner patterns to technology LEF files. A corner not matched here will not be supported by OpenRCX in the default flow.                                                                                                                                                  | `None`                                                                                        |       |
| `TIME_DERATING_CONSTRAINT`  [PDK]     | Decimal                                                      | Specifies a derating factor to multiply the path delays with. It specifies the upper and lower ranges of timing.                                                                                                                                                                 | `None`                                                                                        | %     |
| `TRISTATE_CELLS`  [PDK]               | List[str]?                                                   | A list of cell names or wildcards of tri-state buffers.                                                                                                                                                                                                                          | `None`                                                                                        |       |
| `VDD_NETS`                            | List[str]?                                                   | Specifies the power nets/pins to be used when creating the power grid for the design.                                                                                                                                                                                            | `None`                                                                                        |       |
| `VDD_PIN`  [PDK]                      | str                                                          | The power pin for the cells.                                                                                                                                                                                                                                                     | `None`                                                                                        |       |
| `WELLTAP_CELL`  [PDK]                 | str?                                                         | Defines the cell used for tap insertion. If not defined, steps should not attempt to insert welltap cells.                                                                                                                                                                       | `None`                                                                                        |       |



### Checker Step-Specific Variables
These variables configure the steps within the `Checker` module.


| Variable Name | Type | Description | Default | Units |
| - | - | - | - | - |
| `ERROR_ON_DISCONNECTED_PINS`  | bool | Checks for disconnected instance pins after detailed routing and quits immediately if so. | `True` |  |
| `ERROR_ON_ILLEGAL_OVERLAPS`  | bool | Checks for illegal overlaps during Magic extraction. In some cases, these imply existing undetected shorts in the design. It raises an error at the end of the flow if so. | `True` |  |
| `ERROR_ON_KLAYOUT_ANTENNA`  | bool | Checks for antenna violations after KLayout antenna check is executed and exits the flow if any was found. | `True` |  |
| `ERROR_ON_KLAYOUT_DENSITY`  | bool | Checks for density violations after KLayout density check is executed and exits the flow if any was found. | `True` |  |
| `ERROR_ON_KLAYOUT_DRC`  | bool | Checks for DRC violations after KLayout DRC is executed and exits the flow if any was found. | `True` |  |
| `ERROR_ON_LINTER_ERRORS`  | bool | Quit immediately on any linter errors. | `True` |  |
| `ERROR_ON_LINTER_TIMING_CONSTRUCTS`  | bool | Quit immediately on any discovered timing constructs during linting. | `True` |  |
| `ERROR_ON_LINTER_WARNINGS`  | bool | Raise an error immediately on any linter warnings. | `False` |  |
| `ERROR_ON_LONG_WIRE`  | bool | Checks if any wire length exceeds the threshold set in the PDK. If so, an error is raised at the end of the flow. | `True` |  |
| `ERROR_ON_LVS_ERROR`  | bool | Checks for LVS errors after Netgen is executed. If any exist, it raises an error at the end of the flow. | `True` |  |
| `ERROR_ON_MAGIC_DRC`  | bool | Checks for DRC violations after magic DRC is executed and exits the flow if any was found. | `True` |  |
| `ERROR_ON_NL_ASSIGN_STATEMENTS`  | bool | Whether to emit an error or simply warn about the existence | `True` |  |
| `ERROR_ON_PDN_VIOLATIONS`  | bool | Checks for unconnected nodes in the power grid. If any exists, an error is raised at the end of the flow. | `True` |  |
| `ERROR_ON_SYNTH_CHECKS`  | bool | Quits the flow immediately if one or more synthesis check errors are flagged. This checks for combinational loops and/or wires with no drivers. | `True` |  |
| `ERROR_ON_TR_DRC`  | bool | Checks for DRC violations after routing and exits the flow if any was found. | `True` |  |
| `ERROR_ON_UNMAPPED_CELLS`  | bool | Checks for unmapped cells after synthesis and quits immediately if so. | `True` |  |
| `ERROR_ON_XOR_ERROR`  | bool | Checks for geometric differences between the Magic and KLayout stream-outs. If any exist, raise an error at the end of the flow. | `True` |  |
| `HOLD_VIOLATION_CORNERS`  [PDK] | List[str]? | A list of wildcards matching IPVT corners to use during checking for hold violations. | `['*']` |  |
| `MAX_CAP_VIOLATION_CORNERS`  [PDK] | List[str]? | A list of wildcards matching IPVT corners to use during checking for max cap violations. | `['']` |  |
| `MAX_SLEW_VIOLATION_CORNERS`  [PDK] | List[str]? | A list of wildcards matching IPVT corners to use during checking for max slew violations. | `['']` |  |
| `SETUP_VIOLATION_CORNERS`  [PDK] | List[str]? | A list of wildcards matching IPVT corners to use during checking for setup violations. | `None` |  |
| `TIMING_VIOLATION_CORNERS`  [PDK] | List[str] | A list of wildcards matching IPVT corners to use during checking for timing violations. | `None` |  |
| `WIRE_LENGTH_THRESHOLD`  [PDK] | Decimal? | A value above which wire lengths generate warnings. | `None` | µm |



### KLayout Step-Specific Variables
These variables configure the steps within the `KLayout` module.


| Variable Name | Type | Description | Default | Units |
| - | - | - | - | - |
| `KLAYOUT_ANTENNA_OPTIONS`  [PDK] | Dict[str, (bool｜int｜str)]? | Options passed directly to the KLayout density runset. They vary from one PDK to another. | `None` |  |
| `KLAYOUT_ANTENNA_RUNSET`  [PDK] | Path? | A path to KLayout antenna runset. | `None` |  |
| `KLAYOUT_CONFLICT_RESOLUTION`  | 'AddToCell'｜ 'OverwriteCell'｜ 'RenameCell'｜ 'SkipNewCell' | Specifies the conflict resolution if a cell name conflict arises. | `RenameCell` |  |
| `KLAYOUT_DEF_LAYER_MAP`  [PDK] | Path | A path to the KLayout LEF/DEF layer mapping (.map) file. | `None` |  |
| `KLAYOUT_DENSITY_OPTIONS`  [PDK] | Dict[str, (bool｜int｜str)]? | Options passed directly to the KLayout density runset. They vary from one PDK to another. | `None` |  |
| `KLAYOUT_DENSITY_RUNSET`  [PDK] | Path? | A path to KLayout density runset. | `None` |  |
| `KLAYOUT_DENSITY_THREADS`  | int? | Specifies the number of threads to be used in KLayout density check.If unset, this will be equal to your machine's thread count. | `None` |  |
| `KLAYOUT_DRC_OPTIONS`  [PDK] | Dict[str, (bool｜int｜str)]? | Options passed directly to the KLayout DRC runset. They vary from one PDK to another. | `None` |  |
| `KLAYOUT_DRC_RUNSET`  [PDK] | Path? | A path to KLayout DRC runset. | `None` |  |
| `KLAYOUT_DRC_THREADS`  | int? | Specifies the number of threads to be used in KLayout DRC.If unset, this will be equal to your machine's thread count. | `None` |  |
| `KLAYOUT_EDITOR_MODE`  | bool | Whether to run the KLayout GUI in editor mode or in viewer mode. | `False` |  |
| `KLAYOUT_FILLER_OPTIONS`  [PDK] | Dict[str, (bool｜int｜str)]? | Options passed directly to the KLayout filler script. They vary from one PDK to another. | `None` |  |
| `KLAYOUT_FILLER_SCRIPT`  [PDK] | Path? | A path to KLayout filler script. | `None` |  |
| `KLAYOUT_GUI_USE_GDS`  | bool | Whether to prioritize GDS (if found) when running this step. | `True` |  |
| `KLAYOUT_LVS_OPTIONS`  [PDK] | Dict[str, (bool｜int｜str)]? | Options passed directly to the KLayout LVS script. They vary from one PDK to another. | `None` |  |
| `KLAYOUT_LVS_SCRIPT`  [PDK] | Path? | A path to KLayout LVS script. | `None` |  |
| `KLAYOUT_PROPERTIES`  [PDK] | Path | A path to the KLayout layer properties (.lyp) file. | `None` |  |
| `KLAYOUT_RENDER_BACKGROUND_COLOR`  | 'white'｜ 'black' | The background color of the image. | `white` |  |
| `KLAYOUT_RENDER_GRID_VISBLE`  | bool | Render the grid in the image. | `False` |  |
| `KLAYOUT_RENDER_OVERSAMPLING`  | int | The oversampling factor (1..3), or 0 for disabling oversampling. | `0` |  |
| `KLAYOUT_RENDER_RESOLUTION`  | int | The horizontal resolution of the image in pixel. | `1000` |  |
| `KLAYOUT_RENDER_SHOW_RULER`  | bool | Enable the ruler in the image. | `False` |  |
| `KLAYOUT_RENDER_TEXT_VISIBLE`  | bool | Enable text in the image. | `False` |  |
| `KLAYOUT_SEALRING_SCRIPT`  [PDK] | Path? | A path to KLayout seal ring script. | `None` |  |
| `KLAYOUT_TECH`  [PDK] | Path | A path to the KLayout layer technology (.lyt) file. | `None` |  |
| `KLAYOUT_XOR_IGNORE_LAYERS`  [PDK] | List[str]? | KLayout layers to ignore during XOR operations. | `None` |  |
| `KLAYOUT_XOR_THREADS`  | int? | Specifies number of threads used in the KLayout XOR check. If unset, this will be equal to your machine's thread count. | `None` |  |
| `KLAYOUT_XOR_TILE_SIZE`  [PDK] | int? | The tile size to parallelize the XOR process with. | `None` | µm |



### Magic Step-Specific Variables
These variables configure the steps within the `Magic` module.


| Variable Name | Type | Description | Default | Units |
| - | - | - | - | - |
| `CELL_MAGLEFS`  [PDK] | List[Path]? | A list of pre-processed abstract LEF views for cells. Read as a fallback for undefined cells in scripts where cells are black-boxed. | `None` |  |
| `CELL_MAGS`  [PDK] | List[Path]? | A list of pre-processed concrete views for cells. Read as a fallback for undefined cells. | `None` |  |
| `MAGICRC`  [PDK] | Path | A path to the `.magicrc` file which is sourced before running magic in the flow. | `None` |  |
| `MAGIC_CAPTURE_ERRORS`  | bool | Capture errors print by Magic and quit when a fatal error is encountered. Fatal errors are determined heuristically. It is not guaranteed that they are fatal errors. Hence this is function is gated by a variable. This function is needed because Magic does not throw errors. | `True` |  |
| `MAGIC_DEF_LABELS`  | bool | A flag to choose whether labels are read with DEF files or not. From magic docs: "The '-labels' option to the 'def read' command causes each net in the NETS and SPECIALNETS sections of the DEF file to be annotated with a label having the net name as the label text." If LVS fails, try disabling this option. | `False` |  |
| `MAGIC_DEF_NO_BLOCKAGES`  | bool | If set to true, blockages in DEF files are ignored. Otherwise, they are read as sheets of metal by Magic. | `True` |  |
| `MAGIC_DISABLE_CIF_INFO`  | bool | A flag to disable writing Caltech Intermediate Format (CIF) hierarchy and subcell array information to the GDSII file. | `True` |  |
| `MAGIC_DRC_MAGLEFS`  | List[Path]? | A list of pre-processed abstract LEF views for cells. They are read in before the design and act as blackboxes during DRC. | `None` |  |
| `MAGIC_DRC_USE_GDS`  | bool | A flag to choose whether to run the Magic DRC checks on GDS or not. If not, then the checks will be done on the DEF view of the design, which is a bit faster, but may be less accurate as some DEF/LEF elements are abstract. | `True` |  |
| `MAGIC_EXT_ABSTRACT`  | bool | Extracts a SPICE netlist based on black-boxed standard cells and macros (basically, anything with a LEF) rather than transistors. An error will be thrown if both this and `MAGIC_EXT_USE_GDS` is set to ``True``. | `False` |  |
| `MAGIC_EXT_ABSTRACT_CELLS`  | List[str]? | A list of regular expressions which are matched against the cells of a the design. Matches are abstracted (black-boxed) during SPICE extraction. | `None` |  |
| `MAGIC_EXT_SHORT_RESISTOR`  | bool | Enables adding resistors to shorts- resolves LVS issues if more than one top-level pin is connected to the same net, but may increase runtime and break some designs. Proceed with caution. | `False` |  |
| `MAGIC_EXT_UNIQUE`  | 'all'｜ 'notopports'｜ 'noports'｜ 'none' | Runs `extract unique` with the specified option. The default is "all", and "none" disables `extract unique`, allowing connections between separate nets by label in LVS. | `all` |  |
| `MAGIC_EXT_USE_GDS`  | bool | A flag to choose whether to use GDS for spice extraction or not. If not, then the extraction will be done using the DEF/LEF, which is faster. | `False` |  |
| `MAGIC_FEEDBACK_CONVERSION_THRESHOLD`  | int | If Magic provides more feedback items than this threshold, conversion to KLayout databases is skipped (as something has gone horribly wrong.) | `10000` |  |
| `MAGIC_FILLER_OPTIONS`  [PDK] | List[str]? | Options passed directly to the magic filler script. | `None` |  |
| `MAGIC_FILLER_SCRIPT`  [PDK] | Path? | Path to the magic filler script. | `None` |  |
| `MAGIC_GDS_FLATGLOB`  | List[str]? | Flatten cells by name pattern on input. May be used to avoid false positive DRC errors. The strings may use standard shell-type glob patterns, with * for any length string match, ? for any single character match, \ for special characters, and [] for matching character sets or ranges. | `None` |  |
| `MAGIC_GDS_MERGE`  | bool | A flag to enable merging of connected tiles into polygons during gds write. From magic docs: "Depending on the tile geometry, this may make the output file up to four times smaller, at the cost of speed in generating the output file." | `True` |  |
| `MAGIC_GDS_POLYGON_SUBCELLS`  | bool | A flag to enable polygon subcells in magic for gds read potentially speeding up magic. From magic docs: "Put non-Manhattan polygons. This prevents interations with other polygons on the same plane and so reduces tile splitting." | `False` |  |
| `MAGIC_GUI_USE_GDS`  | bool | Whether to prioritize GDS (if found) when running this step. | `True` |  |
| `MAGIC_INCLUDE_GDS_POINTERS`  | bool | A flag to choose whether to include GDS pointers in the generated mag files or not. | `False` |  |
| `MAGIC_LEF_WRITE_USE_GDS`  | bool | A flag to choose whether to use GDS for LEF writing. If not, then the extraction will be done using abstract LEF views. | `False` |  |
| `MAGIC_MACRO_STD_CELL_SOURCE`  | 'PDK'｜ 'macro' | If set to PDK, magic will use the PDK definition of the STD cells for macros inside the design. Otherwise, the macro is completely treated as a blackbox and magic will use the existing cell definition inside the macro gds. This mode is only supported for macros specified in MACROS variable | `macro` |  |
| `MAGIC_PDK_SETUP`  [PDK] | Path | A path to a PDK-specific setup file sourced by `.magicrc`. | `None` |  |
| `MAGIC_RCX_CTHRESH`  | float | Capacitance threshold value. | `0.1` | femtofarads |
| `MAGIC_RCX_DO_CAPACITANCE`  | bool | Whether to extract local capacitance values. | `True` |  |
| `MAGIC_RCX_DO_RESISTANCE`  | bool | Whether to perform detailed (i.e. non-lumped) resistance extraction. | `True` |  |
| `MAGIC_RCX_EXTRACT_STYLE`  | str | Capacitance extraction corner. For open PDKs, the options are generally: 'ngspice(lrlc)', low resistance, low capacitance; 'ngspice(hrlc)', high resistance, low capacitance; 'ngspice(lrhc)', low resistance, high capacitance; 'ngspice(hrhc)', high resistance, low capacitance. 'ngspice(hrhc)' is typically the slowest corner, and 'ngspice(lrlc) is typically the fastest corner. Defaults to 'ngspice()', which is the default typical extraction corner. | `ngspice()` |  |
| `MAGIC_TECH`  [PDK] | Path | A path to a Magic tech file which, mainly, has DRC rules. | `None` |  |
| `MAGIC_WRITE_FULL_LEF`  | bool | A flag to specify whether or not the output LEF should include all shapes inside the macro or an abstracted view of the macro LEF view via magic. | `False` |  |
| `MAGIC_WRITE_LEF_PINONLY`  | bool | If true, the LEF write will mark only areas that are port labels as pins, while marking the rest of each related net as an obstruction. Otherwise, the labeled port and the any connected metal on the same layer are marked as a pin. | `False` |  |
| `MAGIC_ZEROIZE_ORIGIN`  | bool | A flag to move the layout such that it's origin in the lef generated by magic is 0,0. | `False` |  |



### Misc Step-Specific Variables
These variables configure the steps within the `Misc` module.

No step-specific variables for this module.

### Netgen Step-Specific Variables
These variables configure the steps within the `Netgen` module.


| Variable Name | Type | Description | Default | 
| - | - | - | - | 
| `LVS_FLATTEN_CELLS`  | List[str]? | A list of cell names to be flattened while running LVS | `None` |
| `LVS_IGNORE_CELLS`  | List[str]? | A list of cell names to be ignored while running LVS | `None` |
| `LVS_INCLUDE_MARCO_NETLISTS`  | bool | A flag that enables including the gate-level netlist of macros while running Netgen | `False` |
| `MAGIC_EXT_USE_GDS`  | bool | A flag to choose whether to use GDS for spice extraction or not. If not, then the extraction will be done using the DEF/LEF, which is faster. | `False` |
| `NETGEN_SETUP`  [PDK] | Path | A path to the setup file for Netgen used to configure LVS. If set to None, this PDK will not support Netgen-based steps. | `None` |



### Odb Step-Specific Variables
These variables configure the steps within the `Odb` module.


| Variable Name | Type | Description | Default | Units |
| - | - | - | - | - |
| `CLOCK_WIRE_RC_LAYERS`  [PDK] | List[str]? | Sets estimated clock wire RC values to the average of these layers'. If you provide more than two, the averages are grouped by preferred routing direction and you must provide at least one layer for each routing direction. | `None` |  |
| `DEDUPLICATE_CORNERS`  | bool | Cull duplicate IPVT corners during PNR, i.e. corners that share the same set of lib files and values for LAYERS_RC and VIAS_R as another corner are not considered outside of STA. | `False` |  |
| `DIODE_ON_PORTS`  | 'none'｜ 'in'｜ 'out'｜ 'both' | Always insert diodes on ports with the specified polarities. | `none` |  |
| `DIODE_PADDING`  | int? | Diode cell padding; increases the width of diode cells during placement checks.. | `None` | sites |
| `DPL_CELL_PADDING`  [PDK] | int | Cell padding value (in sites) for detailed placement. The number will be integer divided by 2 and placed on both sides. Should be <= global placement. | `None` | sites |
| `ERRORS_ON_UNMATCHED_IO`  | 'none'｜ 'unmatched_design'｜ 'unmatched_cfg'｜ 'both' | Controls whether to emit an error in: no situation, when pins exist in the design that do not exist in the config file, when pins exist in the config file that do not exist in the design, and both respectively. `both` is recommended, as the default is only for backwards compatibility with LibreLane 1. | `unmatched_design` |  |
| `FP_DEF_TEMPLATE`  | Path? | Points to the DEF file to be used as a template. | `None` |  |
| `FP_TEMPLATE_COPY_POWER_PINS`  | bool | Whether to *always* copy all power pins from the DEF template to the design. | `False` |  |
| `FP_TEMPLATE_MATCH_MODE`  | 'strict'｜ 'permissive' | Whether to require that the pin set of the DEF template and the design should be identical. In permissive mode, pins that are in the design and not in the template will be excluded, and vice versa. | `strict` |  |
| `GPL_CELL_PADDING`  [PDK] | int | Cell padding value (in sites) for global placement. Used by this step only to emit a warning if it's 0. | `None` | sites |
| `GRT_ADJUSTMENT`  | Decimal | Reduction in the routing capacity of the edges between the cells in the global routing graph for all layers. Values range from 0 to 1.  1 = most reduction, 0 = least reduction. | `0.3` |  |
| `GRT_ALLOW_CONGESTION`  | bool | Allow congestion during global routing | `False` |  |
| `GRT_ANTENNA_REPAIR_DIODE_ONLY`  | bool | Only use antenna diodes to fix antenna violations. Cannot be used in conjunction with GRT_ANTENNA_REPAIR_JUMPER_ONLY. | `False` |  |
| `GRT_ANTENNA_REPAIR_ITERS`  | int | The maximum number of iterations for global antenna repairs. | `3` |  |
| `GRT_ANTENNA_REPAIR_JUMPER_ONLY`  | bool | Only use jumpers to fix antenna violations. Cannot be used in conjunction with GRT_ANTENNA_REPAIR_DIODE_ONLY. | `False` |  |
| `GRT_ANTENNA_REPAIR_MARGIN`  | int | The margin to over fix antenna violations. | `10` | % |
| `GRT_LAYER_ADJUSTMENTS`  [PDK] | List[Decimal] | Layer-specific reductions in the routing capacity of the edges between the cells in the global routing graph, delimited by commas. Values range from 0 through 1. | `None` |  |
| `GRT_MACRO_EXTENSION`  | int | Sets the number of GCells added to the blockages boundaries from macros. A GCell is typically defined in terms of Mx routing tracks. The default GCell size is 15 M3 pitches. | `0` |  |
| `GRT_OVERFLOW_ITERS`  | int | The maximum number of iterations waiting for the overflow to reach the desired value. | `50` |  |
| `HEURISTIC_ANTENNA_THRESHOLD`  [PDK] | Decimal? | A Manhattan distance above which a diode is recommended to be inserted by the heuristic inserter. If not specified, the heuristic algorithm. | `None` | µm |
| `IGNORE_DISCONNECTED_MODULES`  [PDK] | List[str]? | Modules (or cells) to ignore when checking for disconnected pins. | `None` |  |
| `INSERT_ECO_BUFFERS`  | List[{class}`ECOBuffer <librelane.steps.odb.ECOBuffer>`]? | List of buffers to insert | `None` |  |
| `INSERT_ECO_DIODES`  | List[{class}`ECODiode <librelane.steps.odb.ECODiode>`]? | List of sinks to insert diodes for. | `None` |  |
| `IO_PIN_H_EXTENSION`  | Decimal | Extends the horizontal io pins outside of the die by the specified units. | `0` | µm |
| `IO_PIN_H_LAYER`  [PDK] | str | The metal layer on which to place horizontally-aligned (long side parallel with the horizon) pins alongside the east and west edges of the die. | `None` |  |
| `IO_PIN_H_LENGTH`  [PDK] | Decimal? |          The length of the pins with an east or west orientation. If unspecified by a PDK, OpenROAD will use whichever is higher of the following two values:             * The pin width             * The minimum value satisfying the minimum area constraint given the pin width          | `None` | µm |
| `IO_PIN_H_THICKNESS_MULT`  | Decimal | A multiplier for horizontal pin thickness. Base thickness is the pins layer min width. | `2` |  |
| `IO_PIN_ORDER_CFG`  | Path? | Path to a custom pin configuration file. | `None` |  |
| `IO_PIN_V_EXTENSION`  | Decimal | Extends the vertical io pins outside of the die by the specified units. | `0` | µm |
| `IO_PIN_V_LAYER`  [PDK] | str | The metal layer on which to place vertically-aligned (long side perpendicular to the horizon) pins alongside the north and south edges of the die. | `None` |  |
| `IO_PIN_V_LENGTH`  [PDK] | Decimal? |          The length of the pins with a north or south orientation. If unspecified by a PDK, OpenROAD will use whichever is higher of the following two values:             * The pin width             * The minimum value satisfying the minimum area constraint given the pin width          | `None` | µm |
| `IO_PIN_V_THICKNESS_MULT`  | Decimal | A multiplier for vertical pin thickness. Base thickness is the pins layer min width. | `2` |  |
| `LAYERS_RC`  [PDK] | Dict[str, Dict[str, Dict[str, Decimal]]]? | Used during PNR steps, Specific custom resistance and capacitance values for metal layers. For each IPVT corner, a mapping for each metal layer is provided. Each mapping describes custom resistance and capacitance values. Usage of wildcards for specifying IPVT corners is allowed. Units are resistance and capacitance per unit length as defined in the first lib file. | `None` |  |
| `MACRO_PLACEMENT_CFG`  | Path? | Path to an optional override for instance placement instead of the `MACROS` object for compatibility with LibreLane 1. If both are `None`, this step is skipped. | `None` |  |
| `MANUAL_GLOBAL_PLACEMENTS`  | Dict[str, {class}`Instance <librelane.config.variable.Instance>`]? | A dictionary of instances to their global (non-legalized and unfixed) placement location. | `None` |  |
| `PDN_CONNECT_MACROS_TO_GRID`  | bool | Enables the connection of macros to the top level power grid. | `True` |  |
| `PDN_ENABLE_GLOBAL_CONNECTIONS`  | bool | Enables the creation of global connections in PDN generation. | `True` |  |
| `PDN_MACRO_CONNECTIONS`  | List[str]? | Specifies explicit power connections of internal macros to the top level power grid, in the format: regex matching macro instance names, power domain vdd and ground net names, and macro vdd and ground pin names `<instance_name_rx> <vdd_net> <gnd_net> <vdd_pin> <gnd_pin>`. | `None` |  |
| `PDN_OBSTRUCTIONS`  | List[Tuple[str, Decimal, Decimal, Decimal, Decimal]]? | Add routing obstructions to the design before PDN stage. If set to `None`, this step is skipped. Format of each obstruction item is a tuple of: layer name, llx, lly, urx, ury,. | `None` | µm |
| `PL_MAX_DISPLACEMENT_X`  | int | Specifies how far an instance can be moved along the X-axis when finding a site where it can be placed during detailed placement. | `500` | µm |
| `PL_MAX_DISPLACEMENT_Y`  | int | Specifies how far an instance can be moved along the Y-axis when finding a site where it can be placed during detailed placement. | `100` | µm |
| `PL_OPTIMIZE_MIRRORING`  | bool | Specifies whether or not to run an optimize_mirroring pass whenever detailed placement happens. This pass will mirror the cells whenever possible to optimize the design. | `True` |  |
| `PNR_CORNERS`  [PDK] | List[str]? | A list of fully-qualified IPVT corners to use during PnR. If unspecified, the value for `STA_CORNERS` from the PDK will be used. | `None` |  |
| `PNR_SDC_FILE`  | Path? | Specifies the SDC file used during all implementation (PnR) steps | `None` |  |
| `ROUTING_OBSTRUCTIONS`  | List[Tuple[str, Decimal, Decimal, Decimal, Decimal]]? | Add routing obstructions to the design. If set to `None`, this step is skipped. Format of each obstruction item is a tuple of: layer name, llx, lly, urx, ury. | `None` | µm |
| `RT_CLOCK_MAX_LAYER`  | str? | The name of highest layer to be used in routing the clock net. | `None` |  |
| `RT_CLOCK_MIN_LAYER`  | str? | The name of lowest layer to be used in routing the clock net. | `None` |  |
| `SET_RC_VERBOSE`  | bool | If set to true, set_rc commands are echoed. Quite noisy, but may be useful for debugging. | `False` |  |
| `SIGNAL_WIRE_RC_LAYERS`  [PDK] | List[str]? | Sets estimated signal wire RC values to the average of these layers'. If you provide more than two, the averages are grouped by preferred routing direction and you must provide at least one layer for each routing direction. | `None` |  |
| `STA_EXTRA_CORNER_TCL_FILE`  | Path? | Experimental: specifies a additional configuration .tcl file to be called during (PnR) steps. | `None` |  |
| `VERILOG_POWER_DEFINE`  | str? | Specifies the name of the define used to guard power and ground connections in the output Verilog header. | `USE_POWER_PINS` |  |
| `VIAS_R`  [PDK] | Dict[str, Dict[str, Dict[str, Decimal]]]? | Used during PNR steps, Specific custom resistance values for via layers. For each IPVT corner, a mapping for each via layer is provided. Each mapping describes custom resistance values. Usage of wildcards for specifying IPVT corners is allowed. Via resistance is per cut/via with units asdefined in the first lib file. | `None` |  |



### OpenROAD Step-Specific Variables
These variables configure the steps within the `OpenROAD` module.


| Variable Name | Type | Description | Default | Units |
| - | - | - | - | - |
| `BOTTOM_MARGIN_MULT`  | Decimal | The core margin, in multiples of site heights, from the bottom boundary. If `DIEA_AREA` and `CORE_AREA` are set, this variable has no effect. | `4` |  |
| `CLOCK_WIRE_RC_LAYERS`  [PDK] | List[str]? | Sets estimated clock wire RC values to the average of these layers'. If you provide more than two, the averages are grouped by preferred routing direction and you must provide at least one layer for each routing direction. | `None` |  |
| `CORE_AREA`  | Tuple[Decimal, Decimal, Decimal, Decimal]? | Specifies a core area (i.e. die area minus margins) to be used in floorplanning. It must be paired with `DIE_AREA`. | `None` | µm |
| `CTS_APPLY_NDR`  | 'none'｜ 'root_only'｜ 'half'｜ 'full' | Applies 2X spacing non-default rule to clock nets except leaf-level nets following some strategy. There are four strategy options: 'none', 'root_only', 'half', 'full'. | `half` |  |
| `CTS_BALANCE_LEVELS`  | bool? | Attempts to keep a similar number of levels in the clock tree across non-register cells (e.g., clock-gate or inverter). | `None` |  |
| `CTS_CLK_BUFFERS`  [PDK] | List[str] | Defines the list of clock buffer names or buffer name wildcards to be used in CTS. | `None` |  |
| `CTS_CLK_MAX_WIRE_LENGTH`  | Decimal | Specifies the maximum wire length on the clock net. | `0` | µm |
| `CTS_CORNERS`  | List[str]? | Clock tree synthesis step-specific override for PNR_CORNERS. | `None` |  |
| `CTS_DELAY_BUFFER_DERATE_PCT`  | Decimal? | This option balances latencies between macro cells and registers by inserting delay buffersThe value of 100 means all needed delay buffers are inserted | `None` | % |
| `CTS_DISABLE_POST_PROCESSING`  | bool | Specifies whether or not to disable post cts processing for outlier sinks. | `False` |  |
| `CTS_DISTANCE_BETWEEN_BUFFERS`  | Decimal | Specifies the distance between buffers when creating the clock tree. | `0` | µm |
| `CTS_MACRO_CLUSTERING_MAX_DIAMETER`  | Decimal? | Specifies the maximum diameter of the sink cluster for the macro tree. | `None` | µm |
| `CTS_MACRO_CLUSTERING_SIZE`  | int? | Specifies the maximum number of sinks per cluster for the macro tree. | `None` |  |
| `CTS_MAX_CAP`  | Decimal? | Overrides the maximum capacitance CTS characterization will test. If omitted, the capacitance is extracted from the lib information of the buffers in CTS_CLK_BUFFERS. | `None` | pF |
| `CTS_MAX_SLEW`  | Decimal? | Overrides the maximum transition time CTS characterization will test. If omitted, the slew is extracted from the lib information of the buffers in CTS_CLK_BUFFERS. | `None` | ns |
| `CTS_OBSTRUCTION_AWARE`  | bool? | Enables obstruction-aware buffering such that clock buffers are not placed on top of blockages or hard macros. This option may reduce legalizer displacement, leading to better latency, skew or timing QoR. | `None` |  |
| `CTS_ROOT_BUFFER`  [PDK] | str | Defines the cell inserted at the root of the clock tree. Used in CTS. | `None` |  |
| `CTS_SINK_BUFFER_MAX_CAP_DERATE_PCT`  | Decimal? | Controls automatic buffer selection. To favor strong(weak) drive strength buffers use a small(large) value.The value of 100 means no derating of max cap limit | `None` | % |
| `CTS_SINK_CLUSTERING_ENABLE`  | bool | Enables pre-clustering of sinks to create one level of sub-tree before building the H-tree. Each cluster is driven by a buffer which becomes the end point of the H-tree structure. | `True` |  |
| `CTS_SINK_CLUSTERING_MAX_DIAMETER`  | Decimal? | Specifies the maximum diameter of the sink cluster. | `None` | µm |
| `CTS_SINK_CLUSTERING_SIZE`  | int? | Specifies the maximum number of sinks per cluster. | `None` |  |
| `DEDUPLICATE_CORNERS`  | bool | Cull duplicate IPVT corners during PNR, i.e. corners that share the same set of lib files and values for LAYERS_RC and VIAS_R as another corner are not considered outside of STA. | `False` |  |
| `DESIGN_REPAIR_BUFFER_INPUT_PORTS`  | bool | Specifies whether or not to insert buffers on input ports when design repairs are run. | `True` |  |
| `DESIGN_REPAIR_BUFFER_OUTPUT_PORTS`  | bool | Specifies whether or not to insert buffers on output ports when design repairs are run. | `True` |  |
| `DESIGN_REPAIR_MAX_CAP_PCT`  | Decimal | Specifies a margin for the capacitances during design repair. | `20` | % |
| `DESIGN_REPAIR_MAX_SLEW_PCT`  | Decimal | Specifies a margin for the slews during design repair. | `20` | % |
| `DESIGN_REPAIR_MAX_WIRE_LENGTH`  | Decimal | Specifies the maximum wire length cap used by resizer to insert buffers during design repair. If set to 0, no buffers will be inserted. | `0` | µm |
| `DESIGN_REPAIR_REMOVE_BUFFERS`  | bool | Invokes OpenROAD's remove_buffers command to remove buffers from synthesis, which gives OpenROAD more flexibility when buffering nets. | `False` |  |
| `DESIGN_REPAIR_TIE_FANOUT`  | bool | Specifies whether or not to repair tie cells fanout when design repairs are run. | `True` |  |
| `DESIGN_REPAIR_TIE_SEPARATION`  | bool | Allows tie separation when performing design repairs. | `False` |  |
| `DIODE_PADDING`  | int? | Diode cell padding; increases the width of diode cells during placement checks.. | `None` | sites |
| `DPL_CELL_PADDING`  [PDK] | int | Cell padding value (in sites) for detailed placement. The number will be integer divided by 2 and placed on both sides. Should be <= global placement. | `None` | sites |
| `DRT_ANTENNA_REPAIR_DIODE_ONLY`  | bool | Only use antenna diodes to fix antenna violations. Cannot be used in conjunction with DRT_ANTENNA_REPAIR_JUMPER_ONLY. | `False` |  |
| `DRT_ANTENNA_REPAIR_ITERS`  | int | The maximum number of iterations to run antenna repair. Set to a positive integer to attempt to repair antennas and then re-run DRT as appropriate. | `3` |  |
| `DRT_ANTENNA_REPAIR_JUMPER_ONLY`  | bool | Only use jumpers to fix antenna violations. Cannot be used in conjunction with DRT_ANTENNA_REPAIR_DIODE_ONLY. | `False` |  |
| `DRT_ANTENNA_REPAIR_MARGIN`  | int | The margin to over fix antenna violations. | `10` | % |
| `DRT_ASSIGN_NDR`  | dict[str, str]? | Specify which nets should be assigned to which non-default rule. The net name is a regular expression. Use '^name\$' to match an exact name. | `None` |  |
| `DRT_OPT_ITERS`  | int | Specifies the maximum number of optimization iterations during Detailed Routing in TritonRoute. | `64` |  |
| `DRT_SAVE_DRC_REPORT_ITERS`  | int? | Write a DRC report every N iterations. If DRT_SAVE_SNAPSHOTS is enabled, there is an implicit default value of 1. | `None` |  |
| `DRT_SAVE_SNAPSHOTS`  | bool | Experimental: saves an odb snapshot of the layout each routing iteration. This increases disk usage considerably but is useful for debugging. | `False` |  |
| `DRT_THREADS`  | int? | Specifies the number of threads to be used in OpenROAD Detailed Routing. If unset, this will be equal to your machine's thread count. | `None` |  |
| `EXTRA_SITES`  [PDK] | List[str]? | Explicitly specify sites other than `PLACE_SITE` to create rows for. If the alternate-site standard cells properly declare the `SITE` property, you do not need to provide this explicitly. | `None` |  |
| `EXTRA_SPEFS`  | List[(str｜Path)]? | A variable that only exists for backwards compatibility with LibreLane <2.0.0 and should not be used by new designs. | `None` |  |
| `FP_ASPECT_RATIO`  | Decimal | The core's aspect ratio (height / width). | `1` |  |
| `FP_CORE_UTIL`  | Decimal | The core utilization percentage. | `50` | % |
| `FP_DEF_TEMPLATE`  | Path? | Points to the DEF file to be used as a template. | `None` |  |
| `FP_FLIP_SITES`  [PDK] | List[str]? | Flip these sites vertically. Useful in niche alignment scenarios where single-height cells have ground at the south side and double-height cells have power at the south side, causing a short. In that situation, flipping the sites for single-height cells resolves the issue. | `None` |  |
| `FP_MACRO_HORIZONTAL_HALO`  | Decimal | Specify the horizontal halo size around macros. | `10` | µm |
| `FP_MACRO_VERTICAL_HALO`  | Decimal | Specify the vertical halo size around macros. | `10` | µm |
| `FP_OBSTRUCTIONS`  | List[Tuple[Decimal, Decimal, Decimal, Decimal]]? | Obstructions applied at floorplanning stage. Placement sites are never generated at these locations, which guarantees that it will remain empty throughout the entire flow. | `None` | µm |
| `FP_PRUNE_THRESHOLD`  [PDK] | Decimal? | If specified, all rows smaller in width than this value will be removed. This helps avoid "islets" of cells that are hard to route and connect to PDNs. | `None` | µm |
| `FP_SIZING`  | 'absolute'｜ 'relative' | Sizing mode for floorplanning | `relative` |  |
| `FP_TAPCELL_DIST`  [PDK] | Decimal? | The distance between tap cell columns. Must be specified if WELLTAP_CELL is specified. | `None` | µm |
| `FP_TRACKS_INFO`  [PDK] | Path | A path to the a classic OpenROAD `.tracks` file. Used by the floorplanner to generate tracks. | `None` |  |
| `GPL_CELL_PADDING`  [PDK] | int | Cell padding value (in sites) for global placement. The number will be integer divided by 2 and placed on both sides. | `None` | sites |
| `GRT_ADJUSTMENT`  | Decimal | Reduction in the routing capacity of the edges between the cells in the global routing graph for all layers. Values range from 0 to 1.  1 = most reduction, 0 = least reduction. | `0.3` |  |
| `GRT_ALLOW_CONGESTION`  | bool | Allow congestion during global routing | `False` |  |
| `GRT_ANTENNA_REPAIR_DIODE_ONLY`  | bool | Only use antenna diodes to fix antenna violations. Cannot be used in conjunction with GRT_ANTENNA_REPAIR_JUMPER_ONLY. | `False` |  |
| `GRT_ANTENNA_REPAIR_ITERS`  | int | The maximum number of iterations for global antenna repairs. | `3` |  |
| `GRT_ANTENNA_REPAIR_JUMPER_ONLY`  | bool | Only use jumpers to fix antenna violations. Cannot be used in conjunction with GRT_ANTENNA_REPAIR_DIODE_ONLY. | `False` |  |
| `GRT_ANTENNA_REPAIR_MARGIN`  | int | The margin to over fix antenna violations. | `10` | % |
| `GRT_DESIGN_REPAIR_MAX_CAP_PCT`  | Decimal | Specifies a margin for the capacitances during design post-grt repair. | `10` | % |
| `GRT_DESIGN_REPAIR_MAX_SLEW_PCT`  | Decimal | Specifies a margin for the slews during post-grt design repair. | `10` | % |
| `GRT_DESIGN_REPAIR_MAX_WIRE_LENGTH`  | Decimal | Specifies the maximum wire length cap used by resizer to insert buffers during post-grt design repair. If set to 0, no buffers will be inserted. | `0` | µm |
| `GRT_DESIGN_REPAIR_RUN_GRT`  | bool | Enables running GRT before and after running resizer | `True` |  |
| `GRT_LAYER_ADJUSTMENTS`  [PDK] | List[Decimal] | Layer-specific reductions in the routing capacity of the edges between the cells in the global routing graph, delimited by commas. Values range from 0 through 1. | `None` |  |
| `GRT_MACRO_EXTENSION`  | int | Sets the number of GCells added to the blockages boundaries from macros. A GCell is typically defined in terms of Mx routing tracks. The default GCell size is 15 M3 pitches. | `0` |  |
| `GRT_OVERFLOW_ITERS`  | int | The maximum number of iterations waiting for the overflow to reach the desired value. | `50` |  |
| `GRT_RESIZER_ALLOW_SETUP_VIOS`  | bool | Allows setup violations when fixing hold. | `False` |  |
| `GRT_RESIZER_FIX_HOLD_FIRST`  | bool | Experimental: attempt to fix hold violations before setup violations, which may lead to better timing results. | `False` |  |
| `GRT_RESIZER_HOLD_MAX_BUFFER_PCT`  | Decimal | Specifies a max number of buffers to insert to fix hold violations. This number is calculated as a percentage of the number of instances in the design. | `50` | % |
| `GRT_RESIZER_HOLD_MAX_UTIL_PCT`  | Decimal? | Defines the percentage of core area used during hold fixing. | `None` | % |
| `GRT_RESIZER_HOLD_REPAIR_TNS_PCT`  | Decimal? | Percentage of violating endpoints to repair during hold fixing. | `None` | % |
| `GRT_RESIZER_HOLD_SLACK_MARGIN`  | Decimal | Specifies a time margin for the slack when fixing hold violations. Normally the resizer will stop when it reaches zero slack. This option allows you to overfix. | `0.05` | ns |
| `GRT_RESIZER_RUN_GRT`  | bool | Gates running global routing after resizer steps. May be useful to disable for designs where global routing takes non-trivial time. | `True` |  |
| `GRT_RESIZER_SETUP_BUFFERING`  | bool | Rebuffering and load splitting during setup fixing. | `True` |  |
| `GRT_RESIZER_SETUP_BUFFER_REMOVAL`  | bool | Buffer removal transform during setup fixing. | `True` |  |
| `GRT_RESIZER_SETUP_GATE_CLONING`  | bool | Enables gate cloning when attempting to fix setup violations | `True` |  |
| `GRT_RESIZER_SETUP_MAX_BUFFER_PCT`  | Decimal | Specifies a max number of buffers to insert to fix setup violations. This number is calculated as a percentage of the number of instances in the design. | `50` | % |
| `GRT_RESIZER_SETUP_MAX_UTIL_PCT`  | Decimal? | Defines the percentage of core area used during setup fixing. | `None` | % |
| `GRT_RESIZER_SETUP_REPAIR_TNS_PCT`  | Decimal? | Percentage of violating endpoints to repair during setup fixing. | `None` | % |
| `GRT_RESIZER_SETUP_SLACK_MARGIN`  | Decimal | Specifies a time margin for the slack when fixing setup violations. | `0.025` | ns |
| `IO_EXCLUDE_PIN_REGION`  | List[str]? | List of regions where pins cannot be placed. The regions are strings in the format `{edge}:{interval}` where edge is `top|bottom|left|right` and the interval is either `*` to exclude the entire edge or `{begin}-{end}` to exclude a part of the edge, where `begin` and `end` are either absolute distance values or themselves `*` to denote the very start or end of an edge. | `None` | µm |
| `IO_PIN_CORNER_AVOIDANCE`  | Decimal? | The distance from each corner within which pin placement should be avoided. | `None` | µm |
| `IO_PIN_H_EXTENSION`  | Decimal | Extends the horizontal io pins outside of the die by the specified units. | `0` | µm |
| `IO_PIN_H_LAYER`  [PDK] | str | The metal layer on which to place horizontally-aligned (long side parallel with the horizon) pins alongside the east and west edges of the die. | `None` |  |
| `IO_PIN_H_LENGTH`  [PDK] | Decimal? |          The length of the pins with an east or west orientation. If unspecified by a PDK, OpenROAD will use whichever is higher of the following two values:             * The pin width             * The minimum value satisfying the minimum area constraint given the pin width          | `None` | µm |
| `IO_PIN_H_THICKNESS_MULT`  | Decimal | A multiplier for horizontal pin thickness. Base thickness is the pins layer min width. | `2` |  |
| `IO_PIN_MIN_DISTANCE`  [PDK] | Decimal? | The minimum distance between two pins. The unit is microns or routing tracks, depending on whether IO_PIN_MIN_DISTANCE_IN_TRACKS is set. If unspecified by a PDK, OpenROAD will use the length of two routing tracks. | `None` | µm or routing tracks |
| `IO_PIN_MIN_DISTANCE_IN_TRACKS`  [PDK] | bool? | Setting this variable to true allows IO_PIN_MIN_DISTANCE to be set in number of tracks instead of microns. | `None` |  |
| `IO_PIN_ORDER_CFG`  | Path? | Path to a custom pin configuration file. | `None` |  |
| `IO_PIN_PLACEMENT_MODE`  | 'matching'｜ 'annealing'｜ 'random_equidistant' | Decides the mode of the random IO placement option. | `matching` |  |
| `IO_PIN_V_EXTENSION`  | Decimal | Extends the vertical io pins outside of the die by the specified units. | `0` | µm |
| `IO_PIN_V_LAYER`  [PDK] | str | The metal layer on which to place vertically-aligned (long side perpendicular to the horizon) pins alongside the north and south edges of the die. | `None` |  |
| `IO_PIN_V_LENGTH`  [PDK] | Decimal? |          The length of the pins with a north or south orientation. If unspecified by a PDK, OpenROAD will use whichever is higher of the following two values:             * The pin width             * The minimum value satisfying the minimum area constraint given the pin width          | `None` | µm |
| `IO_PIN_V_THICKNESS_MULT`  | Decimal | A multiplier for vertical pin thickness. Base thickness is the pins layer min width. | `2` |  |
| `LAYERS_RC`  [PDK] | Dict[str, Dict[str, Dict[str, Decimal]]]? | Used during PNR steps, Specific custom resistance and capacitance values for metal layers. For each IPVT corner, a mapping for each metal layer is provided. Each mapping describes custom resistance and capacitance values. Usage of wildcards for specifying IPVT corners is allowed. Units are resistance and capacitance per unit length as defined in the first lib file. | `None` |  |
| `LEFT_MARGIN_MULT`  | Decimal | The core margin, in multiples of site widths, from the left boundary. If `DIE_AREA` are `CORE_AREA` are set, this variable has no effect. | `12` |  |
| `NON_DEFAULT_RULES`  | dict[str, {class}`NDR <librelane.steps.openroad.NDR>`]? | Specify non-default rules. Can be used to change the width, spacing and vias of a net. | `None` |  |
| `PAD_CFG`  | Path? | A custom pad configuration file. If not provided, the default pad config will be used. | `None` |  |
| `PAD_EAST`  | List[str]? | The pad instance names for the east pad row. | `None` |  |
| `PAD_NORTH`  | List[str]? | The pad instance names for the north pad row. | `None` |  |
| `PAD_SOUTH`  | List[str]? | The pad instance names for the south pad row. | `None` |  |
| `PAD_WEST`  | List[str]? | The pad instance names for the west pad row. | `None` |  |
| `PDN_CFG`  | Path? | A custom PDN configuration file. If not provided, the default PDN config will be used. | `None` |  |
| `PDN_CONNECT_MACROS_TO_GRID`  | bool | Enables the connection of macros to the top level power grid. | `True` |  |
| `PDN_CORE_HORIZONTAL_LAYER`  [PDK] | str? | Defines the horizontal PDN layer for the core ring. Falls back to `PDN_HORIZONTAL_LAYER` if undefined. | `None` |  |
| `PDN_CORE_RING`  | bool | Enables adding a core ring around the design. More details on the control variables in the PDK config documentation. | `False` |  |
| `PDN_CORE_RING_ALLOW_OUT_OF_DIE`  [PDK] | bool | If specified, the ring shapes are allowed to be outside the die boundary. | `True` |  |
| `PDN_CORE_RING_CONNECT_TO_PADS`  [PDK] | bool | If specified, the core side of the pad pins will be connected to the ring. | `False` |  |
| `PDN_CORE_RING_HOFFSET`  [PDK] | Decimal | The offset for the horizontal layer in the core ring of generated power distribution networks. | `None` | µm |
| `PDN_CORE_RING_HSPACING`  [PDK] | Decimal | The spacing for the horizontal layer in the core ring of generated power distribution networks. | `None` | µm |
| `PDN_CORE_RING_HWIDTH`  [PDK] | Decimal | The width for the horizontal layer in the core ring of generated power distribution networks. | `None` | µm |
| `PDN_CORE_RING_VOFFSET`  [PDK] | Decimal | The offset for the vertical layer in the core ring of generated power distribution networks. | `None` | µm |
| `PDN_CORE_RING_VSPACING`  [PDK] | Decimal | The spacing for the vertical layer in the core ring of generated power distribution networks. | `None` | µm |
| `PDN_CORE_RING_VWIDTH`  [PDK] | Decimal | The width for the vertical layer in the core ring of generated power distribution networks. | `None` | µm |
| `PDN_CORE_VERTICAL_LAYER`  [PDK] | str? | Defines the vertical PDN layer for the core ring. Falls back to `PDN_VERTICAL_LAYER` if undefined. | `None` |  |
| `PDN_ENABLE_GLOBAL_CONNECTIONS`  | bool | Enables the creation of global connections in PDN generation. | `True` |  |
| `PDN_ENABLE_PINS`  [PDK] | bool | If specified, the power straps will be promoted to block pins. | `True` |  |
| `PDN_ENABLE_RAILS`  | bool | Enables the creation of rails in the power grid. | `True` |  |
| `PDN_EXTEND_TO`  [PDK] | 'core_ring'｜ 'boundary' | Defines how far the stripes and rings extend. | `core_ring` |  |
| `PDN_HOFFSET`  [PDK] | Decimal | Initial offset for sets of horizontal power straps. | `None` | µm |
| `PDN_HORIZONTAL_HALO`  | Decimal | Sets the horizontal halo around the macros during power grid insertion. | `10` | µm |
| `PDN_HORIZONTAL_LAYER`  [PDK] | str | Defines the horizontal PDN layer. | `None` |  |
| `PDN_HPITCH`  [PDK] | Decimal | Inter-distance (between sets) of horizontal power straps in generated power distribution networks. | `None` | µm |
| `PDN_HSPACING`  [PDK] | Decimal | Intra-spacing (within a set) of horizontal straps in generated power distribution networks. | `None` | µm |
| `PDN_HWIDTH`  [PDK] | Decimal | The strap width for the horizontal layer in generated power distribution networks. | `None` | µm |
| `PDN_MACRO_CONNECTIONS`  | List[str]? | Specifies explicit power connections of internal macros to the top level power grid, in the format: regex matching macro instance names, power domain vdd and ground net names, and macro vdd and ground pin names `<instance_name_rx> <vdd_net> <gnd_net> <vdd_pin> <gnd_pin>`. | `None` |  |
| `PDN_MULTILAYER`  | bool | Controls the layers used in the power grid. If set to false, only the lower layer will be used, which is useful when hardening a macro for integrating into a larger top-level design. | `True` |  |
| `PDN_RAIL_LAYER`  [PDK] | str | Defines the metal layer used for PDN rails. | `None` |  |
| `PDN_RAIL_OFFSET`  [PDK] | Decimal | The offset for the power distribution network rails for first metal layer. | `None` | µm |
| `PDN_RAIL_WIDTH`  [PDK] | Decimal | Defines the width of PDN rails on the `PDN_RAIL_LAYER` layer. | `None` | µm |
| `PDN_SKIPTRIM`  | bool | Enables `-skip_trim` option during pdngen which skips the metal trim step, which attempts to remove metal stubs. | `False` |  |
| `PDN_VERTICAL_HALO`  | Decimal | Sets the vertical halo around the macros during power grid insertion. | `10` | µm |
| `PDN_VERTICAL_LAYER`  [PDK] | str | Defines the vertical PDN layer. | `None` |  |
| `PDN_VOFFSET`  [PDK] | Decimal | Initial offset for sets of vertical power straps. | `None` | µm |
| `PDN_VPITCH`  [PDK] | Decimal | Inter-distance (between sets) of vertical power straps in generated power distribution networks. | `None` | µm |
| `PDN_VSPACING`  [PDK] | Decimal | Intra-spacing (within a set) of vertical straps in generated power distribution networks. | `None` | µm |
| `PDN_VWIDTH`  [PDK] | Decimal | The strap width for the vertical layer in generated power distribution networks. | `None` | µm |
| `PL_KEEP_RESIZE_BELOW_OVERFLOW`  | Decimal? | Only applicable when PL_TIMING_DRIVEN is enabled. When the overflow is below the set value, timing-driven iterations will retain the resizer changes instead of reverting them. Allowed values are 0 to 1. If not set, a nonzero default value from OpenROAD will be used | `None` |  |
| `PL_MAX_DISPLACEMENT_X`  | int | Specifies how far an instance can be moved along the X-axis when finding a site where it can be placed during detailed placement. | `500` | µm |
| `PL_MAX_DISPLACEMENT_Y`  | int | Specifies how far an instance can be moved along the Y-axis when finding a site where it can be placed during detailed placement. | `100` | µm |
| `PL_MAX_PHI_COEFFICIENT`  | Decimal? | Sets a upper bound on the µ_k variable in the GPL algorithm. Useful if global placement diverges.See https://openroad.readthedocs.io/en/latest/main/src/gpl/README.html | `None` |  |
| `PL_MIN_PHI_COEFFICIENT`  | Decimal? | Sets a lower bound on the µ_k variable in the GPL algorithm. Useful if global placement diverges. See https://openroad.readthedocs.io/en/latest/main/src/gpl/README.html | `None` |  |
| `PL_OPTIMIZE_MIRRORING`  | bool | Specifies whether or not to run an optimize_mirroring pass whenever detailed placement happens. This pass will mirror the cells whenever possible to optimize the design. | `True` |  |
| `PL_RESIZER_ALLOW_SETUP_VIOS`  | bool | Allows the creation of setup violations when fixing hold violations. Setup violations are less dangerous as they simply mean a chip may not run at its rated speed, however, chips with hold violations are essentially dead-on-arrival. | `False` |  |
| `PL_RESIZER_FIX_HOLD_FIRST`  | bool | Experimental: attempt to fix hold violations before setup violations, which may lead to better timing results. | `False` |  |
| `PL_RESIZER_HOLD_MAX_BUFFER_PCT`  | Decimal | Specifies a max number of buffers to insert to fix hold violations. This number is calculated as a percentage of the number of instances in the design. | `50` |  |
| `PL_RESIZER_HOLD_MAX_UTIL_PCT`  | Decimal? | Defines the percentage of core area used during hold fixing. | `None` | % |
| `PL_RESIZER_HOLD_REPAIR_TNS_PCT`  | Decimal? | Percentage of violating endpoints to repair during hold fixing. | `None` | % |
| `PL_RESIZER_HOLD_SLACK_MARGIN`  | Decimal | Specifies a time margin for the slack when fixing hold violations. Normally the resizer will stop when it reaches zero slack. This option allows you to overfix. | `0.1` | ns |
| `PL_RESIZER_SETUP_BUFFERING`  | bool | Rebuffering and load splitting during setup fixing. | `True` |  |
| `PL_RESIZER_SETUP_BUFFER_REMOVAL`  | bool | Buffer removal transform during setup fixing. | `True` |  |
| `PL_RESIZER_SETUP_GATE_CLONING`  | bool | Enables gate cloning when attempting to fix setup violations | `True` |  |
| `PL_RESIZER_SETUP_MAX_BUFFER_PCT`  | Decimal | Specifies a max number of buffers to insert to fix setup violations. This number is calculated as a percentage of the number of instances in the design. | `50` | % |
| `PL_RESIZER_SETUP_MAX_UTIL_PCT`  | Decimal? | Defines the percentage of core area used during setup fixing. | `None` | % |
| `PL_RESIZER_SETUP_REPAIR_TNS_PCT`  | Decimal? | Percentage of violating endpoints to repair during setup fixing. | `None` | % |
| `PL_RESIZER_SETUP_SLACK_MARGIN`  | Decimal | Specifies a time margin for the slack when fixing setup violations. | `0.05` | ns |
| `PL_ROUTABILITY_DRIVEN`  | bool | Specifies whether the placer should use routability driven placement. | `True` |  |
| `PL_ROUTABILITY_OVERFLOW_THRESHOLD`  | Decimal? | Sets overflow threshold for routability mode. | `None` |  |
| `PL_SKIP_INITIAL_PLACEMENT`  | bool | Specifies whether the placer should run initial placement or not. | `False` |  |
| `PL_SOFT_OBSTRUCTIONS`  | List[Tuple[Decimal, Decimal, Decimal, Decimal]]? | Soft placement blockages applied at the floorplanning stage. Areas that are soft-blocked will not be used by the initial placer, however, later phases such as buffer insertion or clock tree synthesis are still allowed to place cells in this area. | `None` | µm |
| `PL_TARGET_DENSITY_PCT`  | Decimal? | The desired placement density of cells. If not specified, the value will be equal to (`FP_CORE_UTIL` + 5 * `GPL_CELL_PADDING` + 10). | `None` | % |
| `PL_TIMING_DRIVEN`  | bool | Specifies whether the placer should use timing-driven placement. | `False` |  |
| `PL_WIRE_LENGTH_COEF`  | Decimal | Global placement initial wirelength coefficient. Decreasing the variable will modify the initial placement of the standard cells to reduce the wirelengths | `0.25` |  |
| `PNR_CORNERS`  [PDK] | List[str]? | A list of fully-qualified IPVT corners to use during PnR. If unspecified, the value for `STA_CORNERS` from the PDK will be used. | `None` |  |
| `PNR_SDC_FILE`  | Path? | Specifies the SDC file used during all implementation (PnR) steps | `None` |  |
| `RCX_MERGE_VIA_WIRE_RES`  | bool | If enabled, the via and wire resistances will be merged. | `True` |  |
| `RCX_RULESETS`  [PDK] | Dict[str, Path] | Map of corner patterns to OpenRCX extraction rules. | `None` |  |
| `RCX_SDC_FILE`  | Path? | Specifies SDC file to be used for RCX-based STA, which can be different from the one used for implementation. | `None` |  |
| `RIGHT_MARGIN_MULT`  | Decimal | The core margin, in multiples of site widths, from the right boundary. If `DIE_AREA` are `CORE_AREA` are set, this variable has no effect. | `12` |  |
| `RSZ_CORNERS`  | List[str]? | Resizer step-specific override for PNR_CORNERS. | `None` |  |
| `RSZ_DONT_TOUCH_LIST`  | List[str]? | A list of nets and instances as "don't touch" by design repairs or resizer optimizations. | `None` |  |
| `RSZ_DONT_TOUCH_RX`  | str | A single regular expression designating nets or instances as "don't touch" by design repairs or resizer optimizations. | `\$^` |  |
| `RT_CLOCK_MAX_LAYER`  | str? | The name of highest layer to be used in routing the clock net. | `None` |  |
| `RT_CLOCK_MIN_LAYER`  | str? | The name of lowest layer to be used in routing the clock net. | `None` |  |
| `SET_RC_VERBOSE`  | bool | If set to true, set_rc commands are echoed. Quite noisy, but may be useful for debugging. | `False` |  |
| `SIGNAL_WIRE_RC_LAYERS`  [PDK] | List[str]? | Sets estimated signal wire RC values to the average of these layers'. If you provide more than two, the averages are grouped by preferred routing direction and you must provide at least one layer for each routing direction. | `None` |  |
| `SIGNOFF_SDC_FILE`  | Path? | Specifies the SDC file for STA during signoff | `None` |  |
| `STA_EXTRA_CORNER_TCL_FILE`  | Path? | Experimental: specifies a additional configuration .tcl file to be called during (PnR) steps. | `None` |  |
| `STA_MACRO_PRIORITIZE_NL`  | bool | Prioritize the use of Netlists + SPEF files over LIB files if available for Macros. Useful if extraction was done using OpenROAD, where SPEF files are far more accurate. | `True` |  |
| `STA_MAX_VIOLATOR_COUNT`  | int? | Maximum number of violators to list in violator_list.rpt | `None` |  |
| `STA_THREADS`  | int? | The maximum number of STA corners to run in parallel. If unset, this will be equal to your machine's thread count. | `None` |  |
| `TOP_MARGIN_MULT`  | Decimal | The core margin, in multiples of site heights, from the top boundary. If `DIE_AREA` and `CORE_AREA` are set, this variable has no effect. | `4` |  |
| `VIAS_R`  [PDK] | Dict[str, Dict[str, Dict[str, Decimal]]]? | Used during PNR steps, Specific custom resistance values for via layers. For each IPVT corner, a mapping for each via layer is provided. Each mapping describes custom resistance values. Usage of wildcards for specifying IPVT corners is allowed. Via resistance is per cut/via with units asdefined in the first lib file. | `None` |  |
| `VSRC_LOC_FILES`  | Dict[str, Path]? | Map of power and ground nets to OpenROAD PSM location files. See [this](https://github.com/The-OpenROAD-Project/OpenROAD/tree/master/src/psm#commands) for more info. | `None` |  |



### Verilator Step-Specific Variables
These variables configure the steps within the `Verilator` module.


| Variable Name | Type | Description | Default | 
| - | - | - | - | 
| `LINTER_DEFINES`  | List[str]? | Linter-specific preprocessor definitions; overrides VERILOG_DEFINES for the lint step if exists | `None` |
| `LINTER_DISABLE_WARNINGS`  | List[str]? | Warning codes that are passed to the linter to be disabled. | `['DECLFILENAME', 'EOFNEWLINE']` |
| `LINTER_DISABLE_WARNINGS_BLACKBOX`  | List[str]? | Warning codes that are passed to the linter to be disabled for all blackbox modules. | `['UNDRIVEN', 'UNUSEDSIGNAL']` |
| `LINTER_ERROR_ON_LATCH`  | bool | When a latch is inferred by an `always` block that is not explicitly marked as `always_latch`, report this as a linter error. | `True` |
| `LINTER_ERROR_ON_MULTIDRIVEN`  | bool | When a net has multiple drivers, report this as a linter error. | `True` |
| `LINTER_INCLUDE_PDK_MODELS`  | bool | Include Verilog models of the PDK | `False` |
| `LINTER_RELATIVE_INCLUDES`  | bool | When a file references an include file, resolve the filename relative to the path of the referencing file, instead of relative to the current directory. | `True` |
| `LINTER_VLT`  | Path? | Path to a Verilator Configuration format file (`.vlt`) that is passed to the linter. | `None` |
| `VERILOG_DEFINES`  | List[str]? | Preprocessor defines for input Verilog files | `None` |
| `VERILOG_FILES`  | List[Path] | The paths of the design's Verilog files. | `None` |
| `VERILOG_INCLUDE_DIRS`  | List[Path]? | Specifies the Verilog `include` directories. | `None` |
| `VERILOG_POWER_DEFINE`  | str? | Specifies the name of the define used to guard power and ground connections in the input RTL. | `USE_POWER_PINS` |



### Yosys Step-Specific Variables
These variables configure the steps within the `Yosys` module.


| Variable Name | Type | Description | Default | 
| - | - | - | - | 
| `EQY_FORCE_ACCEPT_PDK`  | bool | Attempt to run EQY even if the PDK's Verilog models are supported by this step. Will likely result in a failure. | `False` |
| `EQY_SCRIPT`  | Path? | The EQY script to use. If unset, a generic EQY script will be generated, but this fails in a number of scenarios. | `None` |
| `GHDL_ARGUMENTS`  | List[str]? | Pass arguments to the ghdl frontend. | `None` |
| `MACRO_PLACEMENT_CFG`  | Path? | This step will warn if this deprecated variable is used, as it indicates Macros are used without the new Macro object. | `None` |
| `SLANG_ARGUMENTS`  | List[str]? | Pass arguments to the Slang frontend. | `None` |
| `SYNTH_ABC_AREA_USE_NF`  | bool | Experimental: uses the &nf delay-based mapper with a very high value instead of the amap area mapper, which may be better in some scenarios at recovering area. | `False` |
| `SYNTH_ABC_BUFFERING`  | bool | Enables `abc` cell buffering. | `False` |
| `SYNTH_ABC_DFF`  | bool | Passes D-flipflop cells through ABC for optimization (which can for example, eliminate identical flip-flops). | `False` |
| `SYNTH_ABC_LEGACY_REFACTOR`  | bool | Replaces the ABC command `drf -l` with `refactor` which matches older versions of LibreLane but is more unstable. | `False` |
| `SYNTH_ABC_LEGACY_REWRITE`  | bool | Replaces the ABC command `drw -l` with `rewrite` which matches older versions of LibreLane but is more unstable. | `False` |
| `SYNTH_ABC_USE_MFS3`  | bool | Experimental: attempts a SAT-based remapping in all area and delay strategies before 'retime', which may improve PPA results. | `False` |
| `SYNTH_ADDER_TYPE`  | 'YOSYS'｜ 'FA'｜ 'RCA'｜ 'CSA' | Adder type to which the \$add and \$sub operators are mapped to.  Possible values are `YOSYS/FA/RCA/CSA`; where `YOSYS` refers to using Yosys internal adder definition, `FA` refers to full-adder structure, `RCA` refers to ripple carry adder structure, and `CSA` refers to carry select adder. | `YOSYS` |
| `SYNTH_AUTONAME`  | bool | Generates names for netlist instances. This results in instance names that can be extremely long, but are more human-readable. | `False` |
| `SYNTH_CHECKS_ALLOW_TRISTATE`  | bool | Ignore multiple-driver warnings if they are connected to tri-state buffers on a best-effort basis. | `True` |
| `SYNTH_CLOCKGATE_MIN_WIDTH`  | int? | If set to a value, a group of flip-flops with size >= SYNTH_CLOCKGATE_MIN_WIDTH and an enable signal are clock-gated instead. | `None` |
| `SYNTH_CLOCKGATE_NEGEDGE_ICG`  [PDK] | str? | The integrated clock gate cell used for positive-edge flip-flops, in the format `<cell>/<active-high clock enable port>/<clk port>/<gated clk port>`. | `None` |
| `SYNTH_CLOCKGATE_POSEDGE_ICG`  [PDK] | str? | The integrated clock gate cell used for positive-edge flip-flops, in the format `<cell>/<active-high clock enable port>/<clk port>/<gated clk port>`. | `None` |
| `SYNTH_CORNER`  [PDK] | str? | A fully qualified IPVT corner to use during synthesis. If unspecified, the value for `DEFAULT_CORNER` from the PDK will be used. | `None` |
| `SYNTH_CSA_MAP`  [PDK] | Path? | A path to a file containing the carry-select adder mapping for Yosys. | `None` |
| `SYNTH_DIRECT_WIRE_BUFFERING`  | bool | Enables inserting buffer cells for directly connected wires. | `True` |
| `SYNTH_ELABORATE_ONLY`  | bool | "Elaborate" the design only without attempting any logic mapping. Useful when dealing with structural Verilog netlists. | `False` |
| `SYNTH_EXTRA_MAPPING_FILE`  | Path? | Points to an extra techmap file for yosys that runs right after yosys `synth` before generic techmap. | `None` |
| `SYNTH_FA_MAP`  [PDK] | Path? | A path to a file containing the full adder mapping for Yosys. | `None` |
| `SYNTH_HIERARCHY_MODE`  | 'flatten'｜ 'deferred_flatten'｜ 'keep' | Affects how hierarchy is maintained throughout and after synthesis. 'flatten' flattens it during and after synthesis. 'deferred_flatten' flattens it after synthesis. 'keep' never flattens it. Please note that when using the Slang plugin, you need to pass '--keep-hierarchy' to `SLANG_ARGUMENTS` separately. To keep the hierarchy partially, use one of the flattening options and set the 'keep_hierarchy' attribute on instances or modules via: `SYNTH_KEEP_HIERARCHY_INSTANCES`, `SYNTH_KEEP_HIERARCHY_MODULES` or `SYNTH_KEEP_HIERARCHY_MIN_COST`. | `flatten` |
| `SYNTH_KEEP_HIERARCHY_INSTANCES`  | List[str]? | A list of instances for which to set the 'keep_hierarchy' attribute. This variable only affects the design when 'flatten' is called through `SYNTH_HIERARCHY_MODE`. | `None` |
| `SYNTH_KEEP_HIERARCHY_MIN_COST`  | int? | Sets the 'keep_hierarchy' attribute on modules where the gate count is estimated to exceed the specified threshold. This prevents larger modules from being flattened. This variable only affects the design when 'flatten' is called through `SYNTH_HIERARCHY_MODE`. | `None` |
| `SYNTH_KEEP_HIERARCHY_MODULES`  | List[str]? | A list of modules for which to set the 'keep_hierarchy' attribute. This variable only affects the design when 'flatten' is called through `SYNTH_HIERARCHY_MODE`. | `None` |
| `SYNTH_LATCH_MAP`  [PDK] | Path? | A path to a file containing the latch mapping for Yosys. | `None` |
| `SYNTH_MUL_BOOTH`  | bool | Runs the booth pass as part of synthesis: See https://yosyshq.readthedocs.io/projects/yosys/en/latest/cmd/booth.html | `False` |
| `SYNTH_MUX4_MAP`  [PDK] | Path? | A path to a file containing the mux4 mapping for Yosys. | `None` |
| `SYNTH_MUX_MAP`  [PDK] | Path? | A path to a file containing the mux mapping for Yosys. | `None` |
| `SYNTH_NORMALIZE_SINGLE_BIT_VECTORS`  | bool | If true, vectors with the shape [0:0] are converted to normal wires in the netlist. If disabled, even one-width pins will be suffixed [0] in the layout when imported by most PnR tools. | `True` |
| `SYNTH_PARAMETERS`  | List[str]? | Key-value pairs to be `chparam`ed in Yosys, in the format `key1=value1`. | `None` |
| `SYNTH_RCA_MAP`  [PDK] | Path? | A path to a file containing the ripple-carry adder mapping for Yosys. | `None` |
| `SYNTH_SHARE_RESOURCES`  | bool | A flag that enables yosys to reduce the number of cells by determining shareable resources and merging them. | `True` |
| `SYNTH_SHOW`  | bool | Generate a graphviz DOT file for the design. This will fail on a completely empty design. | `False` |
| `SYNTH_SIZING`  | bool | Enables `abc` cell sizing (instead of buffering). | `False` |
| `SYNTH_SPLITNETS`  | bool | Splits multi-bit nets into single-bit nets. Easier to trace but may not be supported by all tools. | `True` |
| `SYNTH_STRATEGY`  | 'AREA 0'｜ 'AREA 1'｜ 'AREA 2'｜ 'AREA 3'｜ 'DELAY 0'｜ 'DELAY 1'｜ 'DELAY 2'｜ 'DELAY 3'｜ 'DELAY 4' | Strategies for abc logic synthesis and technology mapping. AREA strategies usually result in a more compact design, while DELAY strategies usually result in a design that runs at a higher frequency. Please note that there is no way to know which strategy is the best before trying them. | `AREA 0` |
| `SYNTH_TIE_UNDEFINED`  | 'high'｜ 'low' | Whether to tie undefined values low or high. Explicitly provide null if you wish to simply leave them undriven. | `low` |
| `SYNTH_TRISTATE_MAP`  [PDK] | Path? | A path to a file containing the tri-state buffer mapping for Yosys. | `None` |
| `SYNTH_WRITE_NOATTR`  | bool | If true, Verilog-2001 attributes are omitted from output netlists. Some utilities do not support attributes. | `True` |
| `USE_SLANG`  | bool | Use the Slang frontend to process files, which has better SystemVerilog parsing capabilities but is not as battle-tested as the default Yosys friend. | `False` |
| `VERILOG_DEFINES`  | List[str]? | Preprocessor defines for input Verilog files. | `None` |
| `VERILOG_FILES`  | List[Path] | The paths of the design's Verilog files. | `None` |
| `VERILOG_INCLUDE_DIRS`  | List[Path]? | Specifies the Verilog `include` directories. | `None` |
| `VERILOG_POWER_DEFINE`  | str? | Specifies the name of the define used to guard power and ground connections in the input RTL. | `USE_POWER_PINS` |
| `VHDL_FILES`  | List[Path] | The paths of the design's VHDL files. | `None` |
| `YOSYS_LOG_LEVEL`  | 'ALL'｜ 'WARNING'｜ 'ERROR' | Which log level for Yosys. At WARNING or higher, the initialization splash is also disabled. | `ALL` |



## 5. Visualizing in Magic (noVNC Desktop)

1. Open a terminal inside the VNC GUI desktop and run:
   ```bash
   magic -T /home/vscode/.ciel/sky130A/libs.tech/magic/sky130A.tech
   ```
2. Type these commands in the **Tkcon console** in order:
   ```tcl
   # 1. Load technology layer/via rules
   lef read /home/vscode/.ciel/sky130A/libs.ref/sky130_fd_sc_hd/techlef/sky130_fd_sc_hd__nom.tlef
   
   # 2. Load standard cell geometries
   lef read /home/vscode/.ciel/sky130A/libs.ref/sky130_fd_sc_hd/lef/sky130_fd_sc_hd.lef
   
   # 3. Load design layout
   def read /home/vscode/Desktop/LibreLane/designs/<design_name>/runs/<RUN_DIR>/final/def/<design_name>.def
   ```
3. Click in the layout window and press **`v`** to zoom to fit.
