# Unreal Engine 5.8 Official MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | Unreal Engine 5.8 Official MCP Server |
| **Category** | Gaming/Simulation |
| **Official** | ⭐ Official (Epic Games / UE 5.8 Native) |
| **Source** | https://dev.epicgames.com/documentation/en-us/unreal-engine |
| **Transport** | stdio |
| **Install** | `Built-in / Enabled via Unreal Editor 5.8 Plugins` |

## Tools & Capabilities
- `spawn_actor` — Spawn static meshes, lights, skeletal meshes, and blueprints directly into active Unreal level
- `modify_lighting` — Adjust directional lights, lumen global illumination, and post-process volume exposure
- `execute_python_command` — Execute remote Python scripts via Unreal Python API and Remote Control
- `run_automation_tests` — Trigger Gauntlet and Unreal Automation test suites with error diagnostics
- `edit_verse_code` — Validate and compile Verse scripts for UEFN (Unreal Editor for Fortnite)

## Client Configuration
```json
{
  "mcpServers": {
    "unreal-engine": {
      "command": "npx",
      "args": [
        "-y",
        "@epicgames/mcp-unreal-engine",
        "--project=/path/to/project.uproject"
      ]
    }
  }
}
```

## Security & Best Practices
- Local socket communication with running Unreal Editor instance. Reversible actor spawning.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
