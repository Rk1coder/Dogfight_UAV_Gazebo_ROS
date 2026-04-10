# ArduPilot Gazebo Plugin & Models

A Gazebo simulation plugin enabling Software-In-The-Loop (SITL) testing for ArduPilot-based vehicles including rovers, multicopters, and fixed-wing aircraft.

---

## Table of Contents

- [Requirements](#requirements)
- [Repository Structure](#repository-structure)
- [Installation](#installation)
- [Usage](#usage)
  - [Launching SITL Simulations](#launching-sitl-simulations)
  - [Ground Control Station (GCS) Setup](#ground-control-station-gcs-setup)
  - [Multi-Vehicle Simulation](#multi-vehicle-simulation)
- [Troubleshooting](#troubleshooting)

---

## Requirements

- **OS:** Ubuntu 16.04 LTS (Xenial) — native installation with full 3D graphics support
- **Gazebo:** Version 7 or later
- **ArduPilot SITL:** Configured and operational

> **Note:** VMware and similar virtual machines do not support full 3D graphics by default. As a workaround, add the following to your `~/.bashrc`:
> ```bash
> echo "export SVGA_VGPU10=0" >> ~/.bashrc
> source ~/.bashrc
> ```
> This enables Gazebo to launch inside a VM, but performance is not guaranteed and depends heavily on available CPU/GPU resources.
> ([Source](http://answers.gazebosim.org/question/13214/virtual-machine-not-launching-gazebo/))

---

## Repository Structure

| Directory | Description |
|-----------|-------------|
| `src/` | Source files for the Gazebo–ArduPilot plugin |
| `include/` | Header files for the Gazebo–ArduPilot plugin |
| `models/` | ArduPilot SITL-compatible vehicle models |
| `models_gazebo/` | Standard Gazebo models from the [OSRF repository](https://bitbucket.org/osrf/gazebo_models/src) |
| `worlds/` | Example Gazebo world files for SITL |

---

## Installation

### 1. Install Gazebo Development Libraries

Choose the version matching your Gazebo installation:

```bash
# Gazebo 7
sudo apt-get install libgazebo7-dev

# Gazebo 8
sudo apt-get install libgazebo8-dev

# Gazebo 9
sudo apt-get install libgazebo9-dev
```

If Gazebo is not yet installed, you can install it via ROS:

```bash
sudo apt install ros-kinetic-desktop-full
```

or directly from [gazebosim.org](http://gazebosim.org/tutorials?tut=install_ubuntu).

### 2. Build and Install the Plugin

```bash
git clone https://github.com/SwiftGust/ardupilot_gazebo
cd ardupilot_gazebo
mkdir build && cd build
cmake ..
make -j4
sudo make install
```

### 3. Configure Environment Variables

Open `~/.bashrc`:

```bash
sudo gedit ~/.bashrc
```

Append the following lines:

```bash
source /usr/share/gazebo/setup.sh

export GAZEBO_MODEL_PATH=~/ardupilot_gazebo/models:${GAZEBO_MODEL_PATH}
export GAZEBO_MODEL_PATH=~/ardupilot_gazebo/models_gazebo:${GAZEBO_MODEL_PATH}
export GAZEBO_RESOURCE_PATH=~/ardupilot_gazebo/worlds:${GAZEBO_RESOURCE_PATH}
export GAZEBO_PLUGIN_PATH=~/ardupilot_gazebo/build:${GAZEBO_PLUGIN_PATH}
```

Apply changes:

```bash
source ~/.bashrc
```

---

## Usage

### Launching SITL Simulations

Each simulation requires two terminals: one for the ArduPilot SITL process and one for Gazebo.

#### Rover

```bash
# Terminal 1 — ArduRover SITL
sim_vehicle.py -v APMrover2 -f gazebo-rover -m --mav10 --map --console -I1

# Terminal 2 — Gazebo
gazebo --verbose rover_ardupilot.world
```

#### Copter (3DR Iris)

```bash
# Terminal 1 — ArduCopter SITL
sim_vehicle.py -v ArduCopter -f gazebo-iris -m --mav10 --map --console -I0

# Terminal 2 — Gazebo
gazebo --verbose iris_ardupilot.world
```

#### Plane (Zephyr Flying Wing)

```bash
# Terminal 1 — ArduPlane SITL
sim_vehicle.py -v ArduPlane -f gazebo-zephyr -m --mav10 --map --console -I0

# Terminal 2 — Gazebo
gazebo --verbose zephyr_ardupilot_demo.world
```

---

### Ground Control Station (GCS) Setup

MAVProxy (used in the commands above via `--map --console`) is the default developer GCS. For a more user-friendly experience, omit those flags and use one of the following:

| GCS | Platform | Download |
|-----|----------|----------|
| APM Planner 2 | Linux/macOS/Windows | [firmware.eu.ardupilot.org](http://firmware.eu.ardupilot.org/Tools/APMPlanner/) |
| QGroundControl | Linux/macOS/Windows | [qgroundcontrol.com](https://donlakeflyer.gitbooks.io/qgroundcontrol-user-guide/en/download_and_install.html) |
| Mission Planner | Windows only | — |

Both APM Planner 2 and QGroundControl connect automatically over UDP when running locally.

---

### Multi-Vehicle Simulation

Each SITL instance uses a unique `-I <N>` index, which determines its UDP port allocation:

| Instance (`-I`) | FDM In Port | FDM Out Port | GCS UDP Port |
|-----------------|-------------|--------------|--------------|
| 0 | 9002 | 9003 | 14550 |
| 1 | 9012 | 9013 | 14560 |
| 2 | 9022 | 9023 | 14570 |

To run multiple vehicles simultaneously, edit your world file to include multiple vehicle models and launch a corresponding SITL instance per vehicle.

> **Important:** Set the `SYSID_THISMAV` ArduPilot parameter to a unique value for each instance to avoid GCS conflicts.

You can use MAVProxy to route telemetry from each instance:

```bash
mavproxy.py --master=udp:localhost:14550 --out=udp:localhost:14551 --out=udp:localhost:14552
mavproxy.py --master=udp:localhost:14560 --out=udp:localhost:14561 --out=udp:localhost:14562
mavproxy.py --master=udp:localhost:14570 --out=udp:localhost:14571 --out=udp:localhost:14572
```

**Example:** [3x IRIS quadcopter simulation](https://www.youtube.com/watch?v=3c7EhVMaqKY&feature=youtu.be) by Jonathan Lopes Florêncio.

---

## Troubleshooting

### `Missing libArduPilotPlugin` on Gazebo launch

This error typically indicates a plugin path mismatch, a known issue with Gazebo 7.

**Steps to resolve:**

1. Verify the plugin installed without errors:
   ```bash
   sudo make install
   ```

2. Confirm the plugin file exists at the install path:
   ```bash
   ls /path/to/install/
   ```

3. Check the expected plugin path from Gazebo's setup script:
   ```bash
   cat /usr/share/gazebo/setup.sh
   ```

4. If the paths differ, copy the plugin manually:
   ```bash
   sudo cp -a /usr/lib/x86_64-linux-gnu/gazebo-7.0/plugins/ /usr/lib/x86_64-linux-gnu/gazebo-7/
   ```

---

> **Disclaimer:** This repository is an active development playground. Some features or configurations may be out of date. Ubuntu 16.04 is assumed throughout.