# DG SDK ROS 2 Bridge

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
