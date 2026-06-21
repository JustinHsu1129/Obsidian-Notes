# Local LibreLane & Magic Setup Notes

  

This notes sheet guides you through starting your local LibreLane GUI server, managing custom designs, running simulation flows, and visualizing results.

  

---

  

## 1. Starting the Server

  

Run the unified startup script from your Mac's host terminal:

  http://localhost:6080/vnc.html

```bash

cd /Users/jhsu2022/Openlane/vsd-openlane

./start-vsd-openlane.sh

```

  

### What this script does:

1. **Auto-Starts Docker**: Launches Docker Desktop on your Mac if it isn't running and waits until it is fully active.

2. **Starts the Server**: Spins up the LibreLane local container with GUI (noVNC) and Docker-in-Docker support.

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

├── config.json (or config.yaml)

├── pin_order.cfg

└── pm32.v

```

* **Auto-Syncing**: The startup script automatically syncs `my_designs/` to the container's path (`~/Desktop/LibreLane/designs/`) every **2 seconds**.

  

---

  

## 3. Running LibreLane Flows

  

Open the terminal **inside your VNC Web Browser Desktop** (at `http://localhost:6080/vnc.html`).

  

### Run Command

Use the `librelane` alias to execute your design:

```bash

librelane ./designs/<design_name>/config.json

```

*Example for pm32:*

```bash

librelane ./designs/pm32/config.json

```

  

*(Note: If you just created the server for the first time, open a **new** terminal tab in VNC to ensure the `librelane` shortcut is active, or run `source ~/.bashrc` in your current tab).*

  

---

  

## 4. Visualizing Layouts with Magic

  

To view layouts, you must run Magic from the terminal **inside the VNC GUI Desktop** (since it requires a graphical interface).

  

### Step 1: Start Magic

Open the terminal in VNC and run:

```bash

magic -T /home/vscode/.ciel/sky130A/libs.tech/magic/sky130A.tech

```

  

### Step 2: Load the Design using Absolute Paths

A Magic layout window and a console window (**Tkcon**) will open. Type the following absolute path commands into the **Tkcon console** to load your design:

  

```tcl

lef read /home/vscode/Desktop/LibreLane/designs/<design_name>/runs/<RUN_DIR>/final/lef/<design_name>.lef

def read /home/vscode/Desktop/LibreLane/designs/<design_name>/runs/<RUN_DIR>/final/def/<design_name>.def

```

*(Replace `<design_name>` with your design name like `pm32`, and `<RUN_DIR>` with the folder name of your latest run, e.g., `RUN_2026-06-21_01-22-30`)*

  

### Step 3: Magic Navigation Shortcuts

Once loaded:

* Click inside the Magic layout window.

* Press **`v`** on your keyboard to auto-fit/zoom the entire routed design onto the screen.

* Hold **Right-Click and drag** to draw a box, then press **`z`** to zoom in.

* Press **`Shift + Z`** to zoom out.