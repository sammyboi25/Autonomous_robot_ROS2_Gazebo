# 🤖 Autonomous Differential Drive Mobile Robot (ROS 2 Jazzy & Gazebo Harmonic)

![ROS 2 Jazzy](https://img.shields.io/badge/ROS2-Jazzy-blue?style=flat-square&logo=ros)
![Gazebo Harmonic](https://img.shields.io/badge/Gazebo-Harmonic-orange?style=flat-square)
![Nav2](https://img.shields.io/badge/Navigation-Nav2-green?style=flat-square)
![License](https://img.shields.io/badge/License-MIT-purple?style=flat-square)

An end-to-end autonomous mobile robot (AMR) platform built using **ROS 2 Jazzy Jalisco** and **Gazebo Harmonic**. This project features a custom $0.5\text{m} \times 0.5\text{m}$ differential-drive robot modeled via URDF/Xacro, spawned into a complex indoor simulation environment (`home.sdf`), localized using Adaptive Monte Carlo Localization (AMCL), and navigated autonomously via the Nav2 stack (DWB local planner and NavFn A* global planner).

---

## The Robot
A square differential drive robot with a 2 driving wheels, a caster wheel, 2D lidar, and a camera
<img width="630" height="469" alt="Screenshot from 2026-10-12 22-31-48" src="https://github.com/user-attachments/assets/0a9f5de8-94ff-45d7-8f55-1ed691918038" />


| Gazebo Harmonic World (`home.sdf`) | RViz2 Navigation & Costmaps |
| :---: | :---: |
| <!-- PLACE YOUR GAZEBO WORLD PHOTO HERE --> <img width="1418" height="639" alt="Screenshot from 2026-10-12 22-33-47" src="https://github.com/user-attachments/assets/d40df4a0-048f-4152-818b-a91162a83639" /> | <!-- PLACE YOUR RVIZ SCREENSHOT HERE --> <img width="1560" height="600" alt="Screenshot from 2026-10-12 22-58-06" src="https://github.com/user-attachments/assets/c82dc293-cc08-413c-846c-09fa1967adc8" /> |

### 🎬 Autonomous Navigation Demo


https://github.com/user-attachments/assets/097effc9-ad3a-4073-8c11-8626ff978b78

---

## 📌 Project Overview

This project implements a complete ROS 2 robotics architecture cleanly split into two modular packages:
1. **`robot_des`**: Contains physical, visual, inertial, and kinematic URDF/Xacro definitions for the robot chassis, wheels, 2D Lidar, and RGB camera sensors.
2. **`robot_bringup`**: Manages environment configuration, ROS-Gazebo message bridging (`ros_gz_bridge`), RViz visualizers, occupancy maps, AMCL localization, and Nav2 path planning execution.

---

## 🛠️ Software Stack & Technologies Used

### Core Framework & Simulation
* **ROS 2 Jazzy Jalisco:** Core robotics middleware managing nodes, topics, services, and action servers.
* **Gazebo Harmonic (`gz-sim`):** Physics engine simulating rigid body dynamics and sensor rendering in the `home.sdf` world.
* **`ros_gz_bridge`:** Parameter bridge translating bi-directional messages between native Gazebo topics (`gz.msgs`) and ROS 2 topics (`sensor_msgs`, `geometry_msgs`, `nav_msgs`).
* **`robot_state_publisher`:** Parses Xacro URDF files to calculate and broadcast 3D coordinate frame transformations (`/tf`).
* **RViz2:** Primary visualizer displaying Lidar point streams, global/local costmaps, localization particle clouds, and active target vectors.

### Navigation & Localization Stack (Nav2)
* **`nav2_amcl`:** Adaptive Monte Carlo Localization configured specifically for differential drive motion kinematics.
* **`nav2_navfn_planner`:** Global path planning using the A* grid search algorithm.
* **`dwb_core` (DWB Local Planner):** Local path execution and trajectory evaluation tuned for square footprint kinematics.
* **`nav2_costmap_2d`:** Dual-layer obstacle representation (Static, Obstacle, and Inflation layers).
* **`nav2_bt_navigator`:** Behavior-tree orchestrator managing task execution and recovery behaviors (`Spin`, `BackUp`).
* **`velocity_smoother` & `collision_monitor`:** Acceleration limits and real-time bounding box safety checks.

---

## 📐 Architecture & Coordinate Frames

### 1. Transform Tree (TF Hierarchy)
The system maintains a spatial tree ensuring all relative frame movements align with the global world origin:

$$\text{map} \xrightarrow[\text{AMCL}]{\text{Localization Drift Fix}} \text{odom} \xrightarrow[\text{Gazebo Diff-Drive}]{\text{Wheel Encoders}} \text{base\_footprint} \xrightarrow[\text{URDF}]{\text{Chassis Center}} \text{base\_link} \xrightarrow[\text{URDF Mount}]{\text{Sensor Position}} \text{lidar\_link}$$

* **`map` $\rightarrow$ `odom`:** Published by **AMCL** to correct cumulative wheel encoder drift.
* **`odom` $\rightarrow$ `base_footprint`:** Published by Gazebo odometry plugins, providing continuous smooth displacement data.
* **`base_footprint` $\rightarrow$ `base_link` $\rightarrow$ `lidar_link`:** Calculated by `robot_state_publisher` based on physical joint origins in `robot.urdf.xacro`.

### 2. ROS 2 $\leftrightarrow$ Gazebo Harmonic Bridge Mapping
Communication between the simulation engine and ROS 2 is configured via `gazebo_bridge.yaml`:

| ROS 2 Topic | Gazebo Topic | Message Type | Direction | Purpose |
| :--- | :--- | :--- | :--- | :--- |
| `/clock` | `/clock` | `rosgraph_msgs/msg/Clock` | GZ $\rightarrow$ ROS | Synchronizes simulation clock |
| `/cmd_vel` | `/cmd_vel` | `geometry_msgs/msg/Twist` | ROS $\rightarrow$ GZ | Motor speed commands |
| `/scan` | `/scan` | `sensor_msgs/msg/LaserScan` | GZ $\rightarrow$ ROS | 2D Lidar range measurements |
| `/tf` | `/model/my_robot/tf` | `tf2_msgs/msg/TFMessage` | GZ $\rightarrow$ ROS | Wheel & model transform poses |
| `/joint_states` | `/joint_states` | `sensor_msgs/msg/JointState` | GZ $\rightarrow$ ROS | Wheel joint angular rotations |
| `/camera/image_raw` | `/camera/image_raw` | `sensor_msgs/msg/Image` | GZ $\rightarrow$ ROS | Camera video feed |

---

## ⚡ Technical Challenges & Solutions

### Gazebo Harmonic Asynchronous Startup & Sensor Detachment
* **The Problem:** In complex environments with multiple 3D mesh assets (`home.sdf` containing beds, tables, and furniture), Gazebo's render engine took several seconds to instantiate mesh geometry. When spawning the robot immediately upon launch, the sensor plugins initialized before joint references registered, causing the Lidar frame to detach from `lidar_joint` and float at the origin plane $(0,0,0)$.
* **The Solution:** Implemented an explicit execution delay in the XML bringup script using `<timer period="5.0">`. Delaying `ros_gz_sim create` allows Gazebo's physics engine and scene broadcaster to stabilize fully before inserting the robot entity.

---

## 🚀 Getting Started & How to Run

### 1. Installation & Dependencies
Ensure ROS 2 Jazzy and Gazebo Harmonic are installed on Ubuntu 24.04 LTS:
```bash
sudo apt update
sudo apt install ros-jazzy-ros-gz-sim ros-jazzy-ros-gz-bridge ros-jazzy-navigation2 ros-jazzy-nav2-bringup -y
```

### 2. Workspace Setup & Build
```bash
# Clone the repository
git clone [https://github.com/sammyboi25/Autonomous_robot_Swayam_ros2.git](https://github.com/sammyboi25/Autonomous_robot_Swayam_ros2.git)
cd Autonomous_robot_Swayam_ros2

# Build and source the workspace
colcon build --symlink-install
source install/setup.bash
```

### 3. Step 1: Launch Simulation
Bring up Gazebo Harmonic with `home.sdf`, spawn the robot, start the topic bridge, and open RViz2:
```bash
ros2 launch robot_bringup sim_launch.xml
```

### 4. Step 2: Launch Autonomous Navigation
In a second terminal, launch the Nav2 navigation stack (AMCL, Costmaps, Planners):
```bash
cd Autonomous_robot_Swayam_ros2
source install/setup.bash
ros2 launch robot_bringup nav_launch.py
```

## 📁 Repository Structure

```text
Autonomous_robot_Swayam_ros2/
├── .gitignore               # Excludes build/, install/, and log/ artifacts
├── LICENSE                  # Open-source license file
├── README.md                # Project documentation
└── src/
    ├── robot_des/           # Kinematics & URDF description package
    │   ├── urdf/
    │   │   └── robot.urdf.xacro # Chassis geometry, links, joints, Lidar/Camera sensors
    │   ├── package.xml
    │   └── setup.py
    │
    └── robot_bringup/       # Simulation bringup & Navigation config package
        ├── config/
        │   ├── gazebo_bridge.yaml # Topic mappings for ros_gz_bridge
        │   └── nav2_params.yaml   # Configs for AMCL, DWB, Costmaps, and BT Navigator
        ├── launch/
        │   ├── sim_launch.xml     # Spawns Gazebo world, robot, bridge, and RViz2
        │   └── nav_launch.py      # Launches Nav2 localization and trajectory stack
        ├── maps/
        │   ├── my_map.pgm         # Occupancy grid image
        │   └── my_map.yaml        # Map metadata and resolution settings
        ├── worlds/
        │   └── home.sdf           # Gazebo Harmonic world file
        ├── package.xml
        └── setup.py
