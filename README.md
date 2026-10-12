# 🤖 Autonomous Differential-Drive Mobile Robot
### ROS 2 Jazzy • Gazebo Harmonic • Nav2

<p align="center">
  <img src="https://img.shields.io/badge/ROS_2-Jazzy-blue?style=flat-square&logo=ros" alt="ROS 2 Jazzy">
  <img src="https://img.shields.io/badge/Gazebo-Harmonic-orange?style=flat-square" alt="Gazebo Harmonic">
  <img src="https://img.shields.io/badge/Navigation-Nav2-green?style=flat-square" alt="Nav2">
  <img src="https://img.shields.io/badge/License-MIT-purple?style=flat-square" alt="MIT License">
</p>

An autonomous mobile robot simulation built using **ROS 2 Jazzy** and **Gazebo Harmonic**. This project brings together robot modeling, physics-based simulation, sensor integration, localization, and autonomous navigation in a custom indoor environment.

The robot uses a **differential-drive configuration** with a 2D LiDAR and an RGB camera. Its URDF/Xacro model is spawned into a custom Gazebo world, while the ROS 2 Navigation Stack (Nav2) enables localization, obstacle-aware path planning, and autonomous movement toward target poses.

The project is organized into two modular ROS 2 packages: `robot_des` for the robot model and `robot_bringup` for simulation, configuration, and navigation.

---

## 📸 Project Demo

### The Robot

A compact, square differential-drive robot featuring two powered wheels, a caster wheel, a 2D LiDAR, and an RGB camera.

<p align="center">
  <img
    src="https://github.com/user-attachments/assets/0a9f5de8-94ff-45d7-8f55-1ed691918038"
    width="450"
    alt="Differential-drive robot model"
  />
</p>

### Gazebo Harmonic — Indoor Simulation

<p align="center">
  <img
    src="https://github.com/user-attachments/assets/d40df4a0-048f-4152-818b-a91162a83639"
    width="450"
    alt="Robot in the Gazebo indoor environment"
  />
</p>

### RViz2 — Navigation and Costmaps

<p align="center">
  <img
    src="https://github.com/user-attachments/assets/c82dc293-cc08-413c-846c-09fa1967adc8"
    width="450"
    alt="RViz2 showing robot navigation and costmaps"
  />
</p>

### 🎬 Autonomous Navigation Demo

Watch the robot navigate through the simulated environment.


https://github.com/user-attachments/assets/60f6ef86-c4a4-4c6a-84e3-45d47261bb7f


---

## ✨ Key Features

- **Custom Robot Modeling:** A differential-drive robot designed using URDF/Xacro, including chassis geometry, wheel joints, and sensor mounting positions.
- **Physics-Based Simulation:** Robot simulation in Gazebo Harmonic using a custom indoor environment.
- **Sensor Integration:** Simulated 2D LiDAR and RGB camera, with sensor data bridged between Gazebo and ROS 2.
- **Robot Localization:** Adaptive Monte Carlo Localization (AMCL) estimates the robot's pose within a known map.
- **Autonomous Navigation:** Nav2 coordinates global path planning, local trajectory planning, and navigation execution.
- **Obstacle-Aware Planning:** Costmaps represent obstacles and support collision-aware navigation.
- **Modular Architecture:** Separate robot-description and bringup packages simplify development and configuration.

---

## 🛠️ Technology Stack

| Component | Technology | Purpose |
|:---|:---|:---|
| Middleware | ROS 2 Jazzy | Nodes, topics, services, and actions |
| Simulation | Gazebo Harmonic | Physics and sensor simulation |
| Robot Model | URDF / Xacro | Links, joints, and geometry |
| Simulation Bridge | `ros_gz_bridge` | Gazebo–ROS 2 communication |
| State Publishing | `robot_state_publisher` | Robot coordinate transforms |
| Visualization | RViz2 | Sensors, maps, and costmaps |
| Localization | AMCL | Robot pose estimation |
| Global Planner | NavFn | Global path planning |
| Local Planner | DWB | Local trajectory control |
| Costmaps | `nav2_costmap_2d` | Obstacle representation |
| Navigation | Nav2 BT Navigator | Navigation task execution |

---

## 🏗️ System Architecture

The project is divided into two ROS 2 packages.

### 1. `robot_des` — Robot Description

Defines the robot's physical and visual model.

Responsibilities include:

- Chassis and wheel geometry.
- Differential-drive wheel joints and caster configuration.
- LiDAR and camera mounting positions.
- URDF/Xacro descriptions used to construct the robot model.

**Main file:** `robot.urdf.xacro`

### 2. `robot_bringup` — Simulation and Navigation

Coordinates the simulation environment and autonomous navigation system.

Responsibilities include:

- Launching the Gazebo world and spawning the robot.
- Configuring the Gazebo–ROS 2 communication bridge.
- Publishing robot state and visualizing the simulation in RViz2.
- Loading occupancy maps and navigation parameters.
- Configuring AMCL, costmaps, planners, and navigation execution.

**Key configuration files:**

- `sim_launch.xml`
- `nav_launch.py`
- `gazebo_bridge.yaml`
- `nav2_params.yaml`

---

## 📐 Coordinate Frames and TF

The robot uses a hierarchical transform tree to represent the relationships between the global map, robot chassis, and sensors.

```text
map
 └── odom
      └── base_footprint
           └── base_link
                ├── lidar_link
                └── camera_link
```

The diagram shows the intended frame hierarchy; the actual published tree depends on the configured Gazebo plugins and URDF/Xacro definitions.

| Transform | Responsibility |
|:---|:---|
| `map → odom` | AMCL corrects the global pose estimate |
| `odom → base_footprint` | Local odometry estimate |
| `base_footprint → base_link` | Ground projection to chassis |
| `base_link → lidar_link` | LiDAR position and orientation |
| `base_link → camera_link` | Camera position and orientation |

`robot_state_publisher` publishes transforms derived from the robot description and joint states. The odometry and localization components provide the corresponding dynamic transforms.

---

## 🔄 Gazebo–ROS 2 Communication

The `gazebo_bridge.yaml` configuration defines communication between Gazebo Harmonic and ROS 2.

| ROS 2 Topic | Message Type | Direction | Purpose |
|:---|:---|:---|:---|
| `/clock` | `Clock` | Gazebo → ROS 2 | Simulation time |
| `/cmd_vel` | `Twist` | ROS 2 → Gazebo | Velocity commands |
| `/scan` | `LaserScan` | Gazebo → ROS 2 | LiDAR measurements |
| `/tf` | `TFMessage` | Gazebo → ROS 2 | Coordinate transforms |
| `/joint_states` | `JointState` | Gazebo → ROS 2 | Joint states |
| `/camera/image_raw` | `Image` | Gazebo → ROS 2 | Camera images |

The exact topic names, message types, and directions depend on the configured bridge mappings and Gazebo plugins.

---

## 🧭 Autonomous Navigation Pipeline

The navigation system combines localization, path planning, obstacle representation, and motion control.

### Localization — AMCL

Adaptive Monte Carlo Localization estimates the robot's pose using LiDAR observations and a known occupancy map.

### Global Planning — NavFn

The NavFn planner computes a path through the map from the robot's estimated position to the navigation goal.

### Local Planning — DWB

The Dynamic Window Approach-based controller evaluates candidate motion trajectories and generates velocity commands to follow the global path while accounting for obstacles and the robot's motion constraints.

### Costmaps and Navigation Execution

The costmaps represent obstacles using map and sensor information. Nav2's behavior tree navigator coordinates the navigation task and configured recovery behaviors.

Together, these components allow the robot to navigate toward target poses in the simulated indoor environment.

---

## 🧩 Technical Challenges and Solutions

### Gazebo Startup Timing and Sensor Placement

**Problem**

During simulation startup, the robot and its sensor plugins must initialize with the correct model and joint references. In a complex world containing multiple meshes, spawning the robot before the simulation is ready can lead to initialization or sensor-frame issues.

**Solution**

An explicit startup delay was introduced in the simulation launch configuration using an XML timer:

```xml
<timer period="5.0">
```

This delays the associated launch action, allowing additional time for the simulation to initialize before the robot is spawned.

The delay is a startup-timing workaround rather than a guarantee that every plugin has finished initializing. A more robust implementation can also check for the required simulation services and entities before spawning the robot.

---

## 🚀 Getting Started

### Prerequisites

- Ubuntu 24.04 LTS
- ROS 2 Jazzy Jalisco
- Gazebo Harmonic
- Colcon build tools
- Required ROS 2 packages for simulation, robot description, and navigation

### 1. Install Dependencies

Install the core simulation and navigation packages:

```bash
sudo apt update

sudo apt install -y \
    ros-jazzy-ros-gz-sim \
    ros-jazzy-ros-gz-bridge \
    ros-jazzy-navigation2 \
    ros-jazzy-nav2-bringup \
    ros-jazzy-xacro \
    ros-jazzy-robot-state-publisher \
    ros-jazzy-rviz2 \
    python3-colcon-common-extensions
```

Make sure ROS 2 Jazzy is installed and sourced according to the official installation instructions.

### 2. Clone the Repository

```bash
git clone https://github.com/sammyboi25/Autonomous_robot_Swayam_ros2.git

cd Autonomous_robot_Swayam_ros2
```

### 3. Build the Workspace

Build the packages from the repository root:

```bash
source /opt/ros/jazzy/setup.bash

colcon build --symlink-install

source install/setup.bash
```

If the build fails because a package dependency is missing, install the dependency and rebuild.

### 4. Launch the Simulation

Start Gazebo Harmonic, spawn the robot, initialize the configured bridge, and open RViz2:

```bash
ros2 launch robot_bringup sim_launch.xml
```

Wait for the simulation and robot to finish initializing.

### 5. Launch Autonomous Navigation

Open a second terminal:

```bash
cd Autonomous_robot_Swayam_ros2

source /opt/ros/jazzy/setup.bash
source install/setup.bash

ros2 launch robot_bringup nav_launch.py
```

Once localization and navigation have initialized, use the configured RViz2 navigation interface to set an initial pose if required and send a navigation goal.

**Note:** Navigation requires a compatible occupancy map and the corresponding map configuration. Ensure that the map and its YAML metadata are correctly configured before launching Nav2.

---

## 📁 Repository Structure

```text
Autonomous_robot_Swayam_ros2/
├── .gitignore
├── LICENSE
├── README.md
└── src/
    ├── robot_des/
    │   ├── urdf/
    │   │   └── robot.urdf.xacro
    │   ├── package.xml
    │   └── setup.py
    │
    └── robot_bringup/
        ├── config/
        │   ├── gazebo_bridge.yaml
        │   └── nav2_params.yaml
        ├── launch/
        │   ├── sim_launch.xml
        │   └── nav_launch.py
        ├── maps/
        │   ├── my_map.pgm
        │   └── my_map.yaml
        ├── worlds/
        │   └── home.sdf
        ├── package.xml
        └── setup.py
```

---

## 📚 Learning Outcomes

Through this project, I worked with:

- Robot modeling using URDF/Xacro.
- ROS 2 package organization and launch files.
- Gazebo Harmonic simulation and ROS 2 integration.
- Sensor communication and coordinate-frame management.
- AMCL-based localization and occupancy maps.
- Nav2 global planning, local trajectory control, and costmaps.
- Debugging simulation startup and robot-model integration issues.

---

## 🔮 Future Improvements

Potential extensions include:

- **SLAM:** Build occupancy maps directly from simulated sensor data.
- **Waypoint Navigation:** Execute multi-goal navigation tasks.
- **Dynamic Obstacle Avoidance:** Evaluate navigation around moving objects.
- **Robot Control:** Tune the differential-drive controller and navigation parameters.
- **Perception:** Integrate camera-based object detection into navigation tasks.

---

## 📄 License

This project is distributed under the MIT License. See the [`LICENSE`](LICENSE) file for details.
