# Blender 3D Modeling & Rendering MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | Blender 3D Modeling & Rendering MCP Server |
| **Category** | Creative/3D |
| **Official** | ⭐ Official / Verified |
| **Source** | https://github.com/blender/mcp-blender |
| **Transport** | stdio |
| **Install** | `npx -y @blender/mcp-server` |

## Tools & Capabilities
- `execute_bpy_script` — Execute Python script in live Blender runtime via `bpy` API
- `inspect_scene_objects` — Query active scene mesh hierarchy, materials, lights, and cameras
- `create_mesh` — Programmatically generate parametric 3D geometry and apply subdivision modifiers
- `render_frame` — Render scene to image or animation frame with Cycles/Eevee engine
- `export_3d_asset` — Export scene objects to glTF, USD, FBX, or OBJ formats

## Client Configuration
```json
{
  "mcpServers": {
    "blender": {
      "command": "npx",
      "args": [
        "-y",
        "@blender/mcp-server",
        "--blender-path=/Applications/Blender.app/Contents/MacOS/Blender"
      ]
    }
  }
}
```

## Security & Best Practices
- Local socket communication with running Blender instance. Validate bpy script bounds.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
