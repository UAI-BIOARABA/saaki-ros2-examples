<div align="center">

<h1> ROS2 Examples for Saaki - Unitree G1 </h1>

<p>
  <a href="README.md">English</a> |
  <a href="README_es.md">Español</a>
</p>

[![ROS 2 Humble](https://img.shields.io/badge/ROS2-Humble-22314E?logo=ros&logoColor=white)](https://docs.ros.org/en/humble/index.html)
[![Ubuntu 22.04](https://img.shields.io/badge/Ubuntu-22.04-E95420?logo=ubuntu&logoColor=white)](https://releases.ubuntu.com/22.04/)
[![C++17](https://img.shields.io/badge/C%2B%2B-17-00599C?logo=c%2B%2B&logoColor=white)](https://isocpp.org/)

[![Robot: Unitree G1](https://img.shields.io/badge/Robot-Unitree%20G1-0A66C2)](https://www.unitree.com/g1)
[![Status: Tested on G1](https://img.shields.io/badge/Status-Tested%20on%20Real%20Hardware-success)](#execution-and-verification)

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
</div>

## 📖 Description

This repository contains a ROS 2 package (`saaki_ros2_examples`) optimized and configured exclusively to control and monitor the **Unitree G1** humanoid robot.

**Credits and Origin:** The example source code and base structure belong to [Unitree Robotics](https://github.com/unitreerobotics/unitree_ros2). This repository is an *adaptation* where the code has been cleaned up by removing scripts from other models (Go2, B2, etc.) and the `CMakeLists.txt` has been restructured to comply with ROS 2 executable installation standards (enabling native use of `ros2 run`). Additionally, we can create custom example scripts while preserving the original ones.

---

## 🛠️ Prerequisites

* **Operating System:** Ubuntu 22.04 LTS
* **ROS Middleware:** ROS 2 Humble
* **Hardware:** Unitree G1 Robot (Ethernet cable connection)

---

## 📦 1. Base Installation (Unitree Dependencies)

Since this package depends on the official robot messages (`unitree_go`, `unitree_hg`, `unitree_api`), it is **mandatory** to install and compile the official Unitree repository as a base layer ("underlay") before compiling this repository.

### 1.1. Install CycloneDDS

The robot communicates through CycloneDDS. In ROS 2 Humble, just install the system binaries:
```bash
sudo apt install ros-humble-rmw-cyclonedds-cpp ros-humble-rosidl-generator-dds-idl libyaml-cpp-dev
```

### 1.2. Clone and compile the official messages
You don't need to compile the entire Unitree repository, just its CycloneDDS workspace:

```bash
# Clone the official repository in your home directory (use our fork)
git clone https://github.com/UAI-BIOARABA/unitree_ros2

# Compile the message packages
cd ~/unitree_ros2/cyclonedds_ws
colcon build
```

---

## 🎁 2. Installation of this Package (Saaki Examples)

Once you have the Unitree base, you can clone and compile this workspace.

```bash
# Create your workspace if you don't have it
mkdir -p ~/ros2_ws/src
cd ~/ros2_ws/src

# Clone this repository (we use '_' instead of '-' following ROS2 standards)
git clone https://github.com/UAI-BIOARABA/saaki-ros2-examples.git saaki_ros2_examples

# Go to the workspace root
cd ~/ros2_ws

# IMPORTANT: Load the Unitree environment BEFORE compiling
source ~/unitree_ros2/setup.sh

# Compile this package
colcon build --symlink-install
```

---

## 🌐 3. Network Configuration (Robot Connection)

For ROS 2 to discover the robot, your PC must be on the same subnet and use CycloneDDS correctly.

### 1. Connect your PC to the robot via Ethernet cable.

### 2. Configure a static IP on your PC:

   - IP: 192.168.123.99

   - Netmask: 255.255.255.0

### 3. Edit the official configuration script (~/unitree_ros2/setup.sh). It should look something like this (change enp44s0 to your network interface name):

```sh
#!/bin/bash
echo "Setup unitree ros2 environment"
source /opt/ros/humble/setup.bash
source $HOME/unitree_ros2/cyclonedds_ws/install/setup.bash
export RMW_IMPLEMENTATION=rmw_cyclonedds_cpp
export CYCLONEDDS_URI='<CycloneDDS><Domain><General><Interfaces>
                            <NetworkInterface name="enp44s0" priority="default" multicast="default" />
                        </Interfaces></General></Domain></CycloneDDS>'
```

---

## 🚀 4. Execution and Verification

Every time you open a new terminal to work with the robot, you must load both environments in this order:

```bash
# 1. Load Unitree dependencies and network configuration
source ~/unitree_ros2/setup.sh

# 2. Load your workspace
source ~/ros2_ws/install/setup.bash
```

### Available Nodes
You can run any of the following nodes using the standard ROS 2 command:

### State Reading:

- ros2 run saaki_ros2_examples read_low_state_hg (Reads the low-level state of motors and sensors).

- ros2 run saaki_ros2_examples read_wireless_controller (Reads the remote control inputs).

### Robot Control (Caution! The robot will move):

- ros2 run saaki_ros2_examples g1_low_level_example (Direct low-level control).

- ros2 run saaki_ros2_examples g1_loco_client_example (High-level locomotion control).

- ros2 run saaki_ros2_examples g1_arm_action_example (Example of arm actions).

- ros2 run saaki_ros2_examples g1_audio_client_example (Audio system test).

*(See the source code of each script for more details about what each example does).*

---

## ⚠️ Common Troubleshooting

- "No executable found" or "Package not found": Make sure you have sourced install/setup.bash in the root of ros2_ws.

- "CMake Error: Could not find unitree_hg": You forgot to run source ~/unitree_ros2/setup.sh before executing colcon build.

- Topics are not appearing (ros2 topic list is empty):

    1. Check that the Ubuntu firewall is disabled (sudo ufw disable).

    2. Verify that your local IP is 192.168.123.99.

    3. Make sure you don't have a ROS_DOMAIN_ID configured that conflicts with the robot's domain ID (by default the robot uses ID 0 or none).

---

## 🧑‍💻 Authors

- **Base code of examples:** [Unitree Robotics](https://github.com/unitreerobotics) &rarr; [unitree_ros2](https://github.com/unitreerobotics/unitree_ros2)
- **Project Manager:** [Juan Fernández](https://github.com/jfbioaraba)
- **Lead Developer:** [Andoni González](https://github.com/andoni92)

---
## Disclaimer

This software and associated materials are provided "as is", without warranty of any kind, either express or implied, including but not limited to warranties of merchantability, fitness for a particular purpose, or non-infringement.

The authors and Bioaraba – Sanitaria Research Institute assume no responsibility for the use, redistribution, or modification of this repository or for any direct or indirect damages arising from its use.

This project is intended exclusively for research and/or educational purposes.
