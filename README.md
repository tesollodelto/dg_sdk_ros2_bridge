> **⚠️ Experimental — ROS 2 Lyrical (work in progress)**
>
> This `lyrical` branch carries in-progress ROS 2 Lyrical compatibility changes
> (modern CMake target linking, and the updated `on_init` hardware-interface API
> in `delto_hardware`). **It has not been fully tested yet.** For stable use on
> ROS 2 Humble / Jazzy, use the `main` branch.

# DG SDK ROS 2 Bridge

[![CI](https://github.com/tesollodelto/dg_sdk_ros2_bridge/actions/workflows/ci.yml/badge.svg)](https://github.com/tesollodelto/dg_sdk_ros2_bridge/actions/workflows/ci.yml)
![ROS 2 Humble](https://img.shields.io/badge/ROS_2-Humble-blue?logo=ros)
![ROS 2 Jazzy](https://img.shields.io/badge/ROS_2-Jazzy-blue?logo=ros)

ROS 2 bridge for the **DG SDK** to interface with Delto grippers.

## Packages

| Package | Description |
|---|---|
| `dg_sdk_ros2_bridge` | SDK bridge node |
| `dg_msgs` | Custom ROS 2 message and service definitions |

## Build

```bash
cd ~/ros2_ws
colcon build --packages-select dg_msgs dg_sdk_ros2_bridge
source install/setup.bash
```
