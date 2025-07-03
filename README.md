# PX4 Offboard Control with ROS 2

## Overview

This tutorial demonstrates how to control a simulated UAV’s velocity using keyboard input through ROS 2 and PX4. It is designed to be beginner-friendly, even for those with no prior experience with ROS 2 or PX4.

This project builds upon the [ARK Electronics ROS2\_PX4\_Offboard\_Example](https://github.com/ARK-Electronics/ROS2_PX4_Offboard_Example), with added updates and patches by me. It streamlines the setup process and updates system compatibility to Ubuntu 24.04 and ROS 2 Jazzy.

## Key Resources

* PX4 ROS2 Offboard Control: [https://docs.px4.io/main/en/ros2/offboard\_control.html](https://docs.px4.io/main/en/ros2/offboard_control.html)
* Original ARK repo: [https://github.com/ARK-Electronics/ROS2\_PX4\_Offboard\_Example](https://github.com/ARK-Electronics/ROS2_PX4_Offboard_Example)
* ROS2 Installation: [https://docs.ros.org/en/humble/Installation/Ubuntu-Install-Debs.html](https://docs.ros.org/en/humble/Installation/Ubuntu-Install-Debs.html)
* Gazebo Harmonic: [https://gazebosim.org/docs/harmonic/install\_ubuntu/](https://gazebosim.org/docs/harmonic/install_ubuntu/)
* General Gazebo docs: [https://gazebosim.org/docs/latest/getstarted](https://gazebosim.org/docs/latest/getstarted)

## Prerequisites

* Ubuntu 24.04 (Noble)
* ROS 2 Jazzy
* Gazebo Harmonic
* PX4 Autopilot
* Micro XRCE-DDS Agent
* Python 3.10+

---

## Step-by-Step Setup

### 1. Install System Dependencies

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y git wget python3-pip python3-jinja2
sudo apt install -y build-essential cmake ninja-build exiftool
sudo apt install -y python3-pyserial python3-toml python3-numpy python3-yaml
```

### 2. Install Python Dependencies

> ⚠️ **Important:** Use `empy==3.3.4` for compatibility with PX4.

```bash
sudo pip3 install empy==3.3.4 pyros-genmsg --break-system-packages
pip3 install kconfiglib jsonschema jinja2 --user
```

---

### 3. Install PX4 Autopilot

```bash
git clone https://github.com/PX4/PX4-Autopilot.git --recursive
bash PX4-Autopilot/Tools/setup/ubuntu.sh
```

After installation, **reboot** your system.

---

### 4. Install ROS 2 Jazzy

Follow the official guide:
[https://docs.ros.org/en/jazzy/Installation/Ubuntu-Install-Debians.html](https://docs.ros.org/en/jazzy/Installation/Ubuntu-Install-Debians.html)

Add this to your `.bashrc` to automatically source ROS 2:

```bash
echo 'source /opt/ros/jazzy/setup.bash' >> ~/.bashrc
source ~/.bashrc
```

---

### 5. Install Gazebo Harmonic

Follow the guide here:
[https://gazebosim.org/docs/harmonic/install\_ubuntu/](https://gazebosim.org/docs/harmonic/install_ubuntu/)

---

### 6. Build Micro XRCE-DDS Agent (PX4 <-> ROS 2 bridge)

```bash
git clone https://github.com/eProsima/Micro-XRCE-DDS-Agent.git
cd Micro-XRCE-DDS-Agent
mkdir build && cd build
cmake ..
make
sudo make install
sudo ldconfig /usr/local/lib/
```

---

## ROS 2 Workspace Setup

### 1. Create Workspace

```bash
mkdir -p ~/sim_env/src
cd ~/sim_env/src
```

### 2. Clone Required Repositories

```bash
# PX4 message definitions
git clone https://github.com/PX4/px4_msgs.git -b release/1.15

# Offboard control example (fork with patches)
git clone https://github.com/AADaoud/PX4_Offboard.git
```

### 3. Build the Workspace

Move out of the `src` directory:

```bash
cd ..
colcon build
```

To rebuild only your own package after making changes:

```bash
colcon build --packages-select px4_offboard
```

Always source the workspace after a build:

```bash
source install/setup.bash
```

You may also want to add this to your `.bashrc` for convenience.

---

## Running the Simulation

Start the full simulation environment:

```bash
ros2 launch px4_offboard offboard_velocity_control.launch.py
```

This launch file does the following:

* Starts the Micro XRCE-DDS Agent (for ROS 2 ↔ PX4 communication)
* Opens Gazebo in a new terminal tab
* Runs the `control.py` node (keyboard teleop)
* Starts `velocity_control.py` (main control logic)
* Opens RViz
* Opens separate terminals for each process using `gnome-terminal`

> Note: The `processes.py` script manages terminal commands. Modify it if you wish to run custom simulations or tools like QGroundControl.

---

## Keyboard Controls

These mimic Mode 2 transmitter inputs:

| Key         | Action         |
| ----------- | -------------- |
| `W`         | Ascend         |
| `S`         | Descend        |
| `A`         | Yaw Left       |
| `D`         | Yaw Right      |
| `↑` (Up)    | Pitch Forward  |
| `↓` (Down)  | Pitch Backward |
| `←` (Left)  | Roll Left      |
| `→` (Right) | Roll Right     |
| `Space`     | Arm / Disarm   |

When armed, the drone will take off and enter offboard mode. Use the controls to navigate. Re-arm with `Space` after landing.

---

## Closing the Simulation

To shut down the simulation cleanly:

1. Go to the **Gazebo** terminal tab.
2. Press `Ctrl + C` to terminate Gazebo and its subprocesses.
3. Close other windows as needed.

Failing to close Gazebo properly may leave background processes that interfere with future runs.

---

## Customizing Launch Behavior

The `processes.py` script launches all subprocesses using `gnome-terminal`. You can customize the simulation environment (e.g., model type or start QGroundControl) by modifying the command strings inside that script.

Example (uncomment to launch QGroundControl):

```python
# "cd ~/QGroundControl && ./QGroundControl.AppImage"
```

---

## Troubleshooting

* **Vehicle won’t arm:** Make sure the PX4 parameter `NAV_DLL_ACT` is set to `0`. You may need to launch QGroundControl to change this setting.
* **Gazebo issues loading:** Ensure you've installed `kconfiglib`, `jsonschema`, and `jinja2`.

---

## Contact & Support

This fork was developed by [AADaoud](https://github.com/AADaoud) based on work from [ARK Electronics](https://github.com/ARK-Electronics/ROS2_PX4_Offboard_Example). For questions or discussion, refer to the original [ARK Discord](https://discord.gg/TDJzJxUMRX) or open an issue on this repository for me to look at.