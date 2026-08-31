# ROS 2 Robot Operating System MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | ROS 2 Robot Operating System MCP Server |
| **Category** | Robotics/Physical-AI |
| **Official** | ⭐ Verified (ROS 2 / Ranch Hand Robotics) |
| **Source** | https://github.com/ranchhandrobotics/ros2-mcp-server |
| **Transport** | stdio |
| **Install** | `python3 -m ros2_mcp_server` |

## Tools & Capabilities
- `list_nodes_and_topics` — Introspect active ROS 2 graph, published topics, subscribed nodes, and message types
- `publish_twist_velocity` — Send `geometry_msgs/Twist` velocity commands to robot base (`/cmd_vel`)
- `call_ros_service` — Trigger ROS 2 services (e.g. sensor calibration, path planning, emergency stop)
- `inspect_tf_tree` — Query coordinate frame transformations between `world`, `base_link`, and `camera_link`

## Client Configuration
```json
{
  "mcpServers": {
    "ros2": {
      "command": "python3",
      "args": [
        "-m",
        "ros2_mcp_server"
      ],
      "env": {
        "ROS_DOMAIN_ID": "0"
      }
    }
  }
}
```

## Security & Best Practices
- Enforce strict velocity limits and emergency stop interlocks on physical robot hardware.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
