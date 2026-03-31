# DG SDK ROS2 Bridge

## Compatibility Notice ⚠️

> **Important:**  
> - The current SDK included in this repository (`libDGSDK.so v1.4.0`) is **not compatible** with recent firmware versions.  
> - **Firmware–SDK Compatibility:**
>   | Model | Minimum Firmware Version | Required SDK Version | Notes |
>   |--------|--------------------------|----------------------|--------|
>   | **B model** | v3.6 or later | **DG SDK v1.6.0+** | Motor direction were revised starting from firmware v3.6.|
>   | **M model** | v2.8 or later | **DG SDK v1.6.0+** | Motor direction were revised starting from firmware v2.8. |
>
> - You can obtain the updated SDK library (`libDGSDK.so`) from Tesollo’s official USB package or distribution.  
>   Replace the existing file in:
>   ```
>   dg_sdk_ros2_bridge/libs/libDGSDK.so
>   ```
>   (Either rename  `libDGSDK_140.so` or `libDGSDK_160.so` → `libDGSDK.so`, or overwrite directly and rebuild your workspace.)

> **Architecture Support:**  
> - The DG SDK currently supports **x86_64 architecture only**.  
> - Running on **ARM64 (aarch64)** platforms such as Jetson or Raspberry Pi is **not supported** and may lead to build or runtime errors.

## Overview
The DG SDK ROS2 Bridge provides a ROS2 interface for the DELTO gripper, allowing Python developers to control the gripper without needing C++ knowledge. This bridge wraps the DG SDK (v1.4.0 included; v1.6.0+ recommended) and exposes its functionality through ROS2 services and messages.

## Features
- **Python-friendly Interface**: Control the gripper entirely through Python using ROS2 services
- **Bridge Architecture**: C++ bridge handles low-level SDK communication while exposing high-level ROS2 interface
- **Multiple Communication Modes**: Supports both serial (USB) and Ethernet (TCP/IP) connections
- **Complete Gripper Control**: System configuration, connection management, and operation control

## Requirements
- ROS2 (tested on Humble and Jazzy)
- DG SDK library (libDGSDK.so v1.4.0 included)
- Python 3
- dg_msgs package (custom messages and services)

## Installation
```bash
# Clone the repository into your ROS2 workspace
cd ~/your_ros2_ws/src
git clone <repository_url>

# Build the package
cd ~/your_ros2_ws
colcon build --packages-select dg_sdk_ros2_bridge dg_msgs

# Source the workspace
source install/setup.bash
```

## Usage

### 1. Launch the Bridge Node
```bash
ros2 run dg_sdk_ros2_bridge dg_sdk_ros2_bridge_node
```

### 2. Python Example
The package includes a complete Python example (`example/dg3fb.py`) demonstrating how to:
- Configure gripper system settings
- Connect to the gripper
- Set gripper options
- Start the system

### 3. Available Services

#### System Configuration
- **Service**: `/dg/set_gripper_system`
- **Type**: `dg_msgs/srv/SetGripperSystem`
- **Purpose**: Configure communication parameters (port, IP, baudrate, etc.)

#### Connection Management
- **Service**: `/dg/connect_to_gripper`
- **Type**: `dg_msgs/srv/ConnectToGripper`
- **Purpose**: Establish connection with the gripper

#### Gripper Options
- **Service**: `/dg/set_gripper_option`
- **Type**: `dg_msgs/srv/SetGripperOption`
- **Purpose**: Configure gripper-specific settings (model, joint count, finger count, etc.)

#### System Control
- **Service**: `/dg/system_start`
- **Type**: `dg_msgs/srv/SystemStart`
- **Purpose**: Start the gripper system operation

### Service Categories

The bridge exposes many services under the `/dg/` namespace. Key services by category:

| Category | Service | Purpose |
|----------|---------|---------|
| **System** | `set_gripper_system` | Configure communication parameters (port, IP, baudrate, mode) |
| **System** | `set_gripper_option` | Set model, joint count, finger count |
| **System** | `system_start` | Start gripper operation |
| **Connection** | `connect_to_gripper` | Establish connection |
| **Connection** | `disconnect_to_gripper` | Disconnect from gripper |
| **Motion** | `move_joint_all` | Move all joints simultaneously |
| **Motion** | `move_joint_finger` | Move all joints in one finger |
| **Motion** | `move_joint` | Move a single joint |
| **Motion** | `move_servo_joint` | Servo (streaming) joint control |
| **Motion** | `move_tcp_all` | Move all fingers in TCP (Cartesian) space |
| **Sensor** | `get_received_gripper_data` | Read current joint positions/efforts |
| **Sensor** | `get_received_fingertip_sensor_data` | Read fingertip F/T sensor data |
| **Sensor** | `set_finger_tip_data_zero` | Zero/calibrate fingertip sensors |
| **Config** | `set_joint_gain_pid` / `set_joint_gain_pid_all` | Set PID gains |
| **Config** | `set_grasp_force` | Set grasping force |
| **Recipe** | `load_recipe_pose_data` | Load a saved pose recipe |
| **Recipe** | `load_recipe_grasp_data` | Load a saved grasp recipe |

### Typical Workflow

A typical session follows this sequence:

```python
# 1. Configure communication
set_gripper_system(ip="169.254.186.72", port=502, communication_mode=0)

# 2. Connect to the gripper
connect_to_gripper()

# 3. Start the system
system_start()

# 4. Move joints
move_joint_all(positions=[...])
```

> **Tip:** Run `ros2 service list | grep dg` to see all available services at runtime.

## Example Code

```python
#!/usr/bin/env python3
import rclpy
from rclpy.node import Node
from dg_msgs.srv import SetGripperSystem, ConnectToGripper
from dg_msgs.msg import GripperSystemSetting

# Initialize ROS2
rclpy.init()
node = Node("gripper_control_node")

# Configure system settings
system_setting = GripperSystemSetting()
system_setting.comport = "/dev/ttyUSB0"  # For serial connection
system_setting.ip = "169.254.186.72"      # For Ethernet connection
system_setting.port = 502
system_setting.communication_mode = 0    # 1: Serial, 0: Ethernet
system_setting.control_mode = 1
system_setting.baudrate = 115200

# Create service client
client = node.create_client(SetGripperSystem, "dg/set_gripper_system")
client.wait_for_service()

# Send configuration request
request = SetGripperSystem.Request()
request.setting = system_setting
future = client.call_async(request)
rclpy.spin_until_future_complete(node, future)

# Check result
result = future.result()
print("Configuration result:", result.result)
```

## Communication Modes

### Serial (USB) Connection
```python
system_setting.communication_mode = 1
system_setting.comport = "/dev/ttyUSB0"  # Adjust to your device
system_setting.baudrate = 115200
```

### Ethernet (TCP/IP) Connection
```python
system_setting.communication_mode = 0
system_setting.ip = "169.254.186.72"  # Gripper IP address
system_setting.port = 502            # Modbus TCP port
```

## Architecture
```
Python Application
       ↓
ROS2 Services/Messages (dg_msgs)
       ↓
C++ Bridge Node (dg_sdk_ros2_bridge_node)
       ↓
DG SDK Library (libDGSDK.so)
       ↓
DELTO Gripper Hardware
```

This architecture allows Python developers to control the gripper without dealing with C++ code or SDK integration details.

## Troubleshooting

### Connection Issues
- Ensure the gripper is powered on and connected
- Check cable connections (USB or Ethernet)
- Verify the correct port/IP settings
- Check permissions for serial port access: `sudo chmod 666 /dev/ttyUSB0`

### Service Not Available
- Ensure the bridge node is running
- Check that the workspace is sourced correctly
- Verify dg_msgs package is built and available

## License
BSD-3-Clause
