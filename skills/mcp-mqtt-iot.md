# MQTT IoT & Edge Messaging MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | MQTT IoT & Edge Messaging MCP Server |
| **Category** | IoT/Messaging |
| **Official** | ⭐ Verified (EMQX / Eclipse MQTT) |
| **Source** | https://github.com/emqx/mcp-mqtt |
| **Transport** | stdio |
| **Install** | `npx -y @emqx/mcp-mqtt` |

## Tools & Capabilities
- `publish_message` — Publish telemetry payloads to designated MQTT topic with QoS levels (0, 1, 2)
- `subscribe_topic` — Listen to sensor data streams and capture incoming JSON payloads
- `list_connected_clients` — Query active edge device connections and last-will-and-testament (LWT) statuses
- `manage_acl_rules` — Inspect and configure topic access control rules for IoT fleets

## Client Configuration
```json
{
  "mcpServers": {
    "mqtt": {
      "command": "npx",
      "args": [
        "-y",
        "@emqx/mcp-mqtt"
      ],
      "env": {
        "MQTT_BROKER_URL": "mqtt://broker.emqx.io:1883",
        "MQTT_USERNAME": "device_user",
        "MQTT_PASSWORD": "device_password"
      }
    }
  }
}
```

## Security & Best Practices
- TLS/SSL broker connection recommended. Restrict publishing permissions to approved topic namespaces.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
