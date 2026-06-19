## Part 1: Comprehensive Step-by-Step Launch Guide

This setup assumes you are working from your Mac terminal inside your `Openlane/vsd-openlane` repository directory, with your PDK already populated in `$HOME/openlane/pdks`.

### Step 1: Start the Graphical noVNC EDA Environment

Instead of running a headless text container, we will launch the unified graphical desktop environment. This command mounts your local designs folder and your downloaded PDK path directly into the container.

Run this command in your **native Mac terminal**:

Bash

```
docker run -d \
  --name osic-gui \
  --platform linux/amd64 \
  -p 6080:80 \
  -v $HOME/openlane/pdks:/foss/pdks:ro \
  -v $(pwd)/.openlane-designs:/foss/designs \
  -e PDK_ROOT=/foss/pdks \
  hpretl/iic-osic-tools:latest
```

### Step 2: Access the Visual Desktop

1. Open your web browser (use an **Incognito / Private Window** to avoid extension conflicts).
    
2. Navigate to: `http://localhost:6080/vnc.html`
    
3. Click **Connect**. If it prompts you for a password, enter: `abc123`
    
4. You will now see a full Linux desktop interface inside your browser tab.
    
5. Use `docker stop osic-gui` to stop the noVNC server if done (use `docker start osic-gui` to restart server). 
    

### Step 3: Open the Terminal Inside noVNC & Launch OpenLane

All subsequent steps happen **inside the web browser terminal application** in noVNC:

1. Click the Terminal icon on the desktop to open a shell interface.
    
2. Launch the OpenLane flow toolchain by running:
    
    Bash
    
    ```
    openlane
    ```
    
    _(This initializes the tool environment and presents you with an interactive OpenLane prompt: `OpenLane Container (1.1.1):/foss/designs%`)_
    

### Step 4: Run the Implementation Flow

To execute a non-interactive automated run on a design (e.g., `picorv32a`), enter this into the OpenLane prompt:

Bash

```
flow.tcl -design /foss/designs/picorv32a -tag test_run
```

### Step 5: Open Magic to View Your Layout GUI

Once the flow finishes running successfully, exit the OpenLane prompt (`exit`) or open a new terminal tab in noVNC, then run:

Bash

```
# 1. Move to the results folder
cd /foss/designs/picorv32a/runs/test_run/results/final/

# 2. Open Magic with the Sky130 technology file
magic -T /foss/pdks/sky130A/libs.tech/magic/sky130A.tech
```

Inside the Magic `tkcon` terminal console window that pops open, load your compiled layout using the absolute paths:

Tcl

```
lef read /foss/designs/picorv32a/runs/test_run/tmp/merged.max.lef
def read /foss/designs/picorv32a/runs/test_run/results/final/def/picorv32a.def
```

_Click into the black layout canvas and press **`v`** to zoom-to-fit your design!_

## Part 2: Working with Custom Designs

When you want to build your own custom digital block instead of using pre-packaged examples like `picorv32a`, you must organize your workspace files intentionally so that OpenLane's automated scripts can find them.

### Where to Place Your Files

On your native host Mac, your design workspace folder is physically mapped to:

`vsd-openlane/.openlane-designs/`

To add a new custom hardware design (e.g., a custom accelerator block named `my_accelerator`), create a dedicated subdirectory structure inside that folder exactly like this:

Plaintext

```
vsd-openlane/.openlane-designs/
└── my_accelerator/
    ├── config.tcl
    └── src/
        ├── my_accelerator.v
        └── my_submodule.v
```

### 1. The `src/` Directory

Put all of your structural or behavioral hardware description source files (`.v` or `.sv`) inside this subdirectory. OpenLane will read these source files during the Yosys logic synthesis stage.

### 2. The `config.tcl` Configuration File

Every custom design requires a `config.tcl` file in its root directory. This file instructs OpenLane how to synthesize and construct your layout. Here is a baseline skeleton configuration you can copy and modify for your designs:

Tcl

```
# Design Name (Must match your top-level Verilog module name)
set ::env(DESIGN_NAME) "my_accelerator"

# Path to Verilog Source Files
set ::env(VERILOG_FILES) [glob $::env(DESIGN_DIR)/src/*.v]

# Clock Configuration
set ::env(CLOCK_PORT) "clk"
set ::env(CLOCK_PERIOD) "10.0"

# Design Core Target Density (Utilization)
set ::env(FP_CORE_UTIL) "35"
set ::env(PL_TARGET_DENSITY) "0.40"

# Handle Die/Core Sizing automatically
set ::env(FP_SIZING) "relative"

# Physical Cell Configurations
set ::env(SYNTH_MAX_FANOUT) "5"
```

### How to Run Your Custom Design

Once your `config.tcl` and `src/*.v` files are saved in place on your Mac, they will immediately show up inside your noVNC Docker container.

To execute your custom design run, simply launch OpenLane as described in Part 1 and call:

Bash

```
flow.tcl -design /foss/designs/my_accelerator -tag run_v1
```