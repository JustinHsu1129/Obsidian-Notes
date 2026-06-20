# Local VSD-OpenLane & Magic Setup Notes

  

This notes sheet guides you through starting your local OpenLane GUI server, managing custom designs, running simulation flows, and visualizing results.

  

---

  

## 1. Starting the Server

  

The main script is fully automated. Run it from your Mac's host terminal:

  

```bash

cd /Users/jhsu2022/Openlane/vsd-openlane

./start-vsd-openlane.sh

```

  

### What this script does:

1. **Auto-Starts Docker**: Launches Docker Desktop on your Mac if it isn't running and waits until it is fully active.

2. **Starts the Server**: Spins up the OpenLane local container with GUI (noVNC) and Docker-in-Docker support.

3. **Real-Time File Syncing**: Begins monitoring and automatically syncing your custom designs into the container.

4. **Auto-Cleanup**: Tails the container logs. **Pressing `Ctrl+C` or closing the terminal window** will automatically stop and remove the container to free up system memory.

  

---

  

## 2. Managing Custom Designs

  

The workflow is set up so you edit files directly on your Mac, and they instantly sync to the container.

  

* **Mac Folder Path**: Place your custom designs inside:

`/Users/jhsu2022/Openlane/vsd-openlane/my_designs/`

*Example Structure:*

```text

my_designs/

└── pm32/

├── config.json (or config.tcl)

├── pin_order.cfg

└── pm32.v

```

* **Auto-Syncing**: The startup script automatically syncs `my_designs/` to the container's path (`~/Desktop/OpenLane/designs/`) every **2 seconds**.

  

---

  

## 3. Running OpenLane Flows

  

Open the terminal **inside your VNC Web Browser Desktop** (at `http://localhost:6080/vnc.html`) or connect via `docker exec -it vsd-openlane-local bash`.

  

### Option A: The Fast Non-Interactive Way (Recommended)

This is the easiest way to run flows. It automatically spins up the inner environment, executes the flow, and overwrites previous runs:

```bash

cd ~/Desktop/OpenLane

make test TEST_DESIGN=<design_name>

```

*Example for pm32:*

```bash

make test TEST_DESIGN=pm32

```

  

### Option B: The Interactive Tcl Console Way

Use this if you want to run steps manually or debug the flow step-by-step:

1. **Mount and enter the OpenLane container**:

```bash

cd ~/Desktop/OpenLane

make mount

```

2. **Prepare the design & start the interactive flow**:

```bash

./flow.tcl -interactive -design <design_name>

```

3. **Run steps in the Tcl console (`%` prompt)**:

* Run the whole flow automatically:

```tcl

run_non_interactive

```

* Or run step-by-step:

```tcl

run_synthesis

run_floorplan

run_placement

run_cts

run_routing

run_magic

```

4. **Exit the environment**:

```tcl

exit

```

Then type `exit` again to leave the container.

  

---

  

## 4. Visualizing Layouts with Magic

  

To view layouts, you must run Magic from the terminal **inside the VNC GUI Desktop** (since it requires a graphical interface).

  

### Step 1: Start Magic

Open the terminal in VNC and run:

```bash

magic -T /home/vscode/.ciel/sky130A/libs.tech/magic/sky130A.tech

```

  

### Step 2: Load the Design using Absolute Paths

A Magic layout window and a console window (**Tkcon**) will open. Type the following absolute path commands into the **Tkcon console** to avoid directory mismatch issues:

  

#### For the `pm32` Design:

```tcl

lef read /home/vscode/Desktop/OpenLane/designs/pm32/runs/<RUN_DIR>/tmp/merged.nom.lef

def read /home/vscode/Desktop/OpenLane/designs/pm32/runs/<RUN_DIR>/results/routing/pm32.def

```

*(Replace `<RUN_DIR>` with the folder name of your latest run, e.g., `RUN_2026.06.20_02.20.31`)*

  

#### For the default `spm` Test Design:

```tcl

lef read /home/vscode/Desktop/OpenLane/designs/spm/runs/openlane_test/tmp/merged.nom.lef

def read /home/vscode/Desktop/OpenLane/designs/spm/runs/openlane_test/results/routing/spm.def

```

  

### Step 3: Magic Navigation Shortcuts

Once loaded:

* Click inside the Magic layout window.

* Press **`v`** on your keyboard to auto-fit/zoom the entire routed design onto the screen.

* Hold **Right-Click** and drag to draw a box, then press **`z`** to zoom in.

* Press **`Shift + Z`** to zoom out.