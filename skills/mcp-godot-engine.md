# Godot Game Engine MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | Godot Game Engine MCP Server |
| **Category** | Gaming/Engine |
| **Official** | ⭐ Verified (Godot Community) |
| **Source** | https://github.com/godotengine/mcp-godot |
| **Transport** | stdio |
| **Install** | `npx -y @godotengine/mcp-server` |

## Tools & Capabilities
- `inspect_scene_tree` — Query active Godot scene nodes, properties, script attachments, and collision shapes
- `edit_gdscript` — Create and refactor GDScript classes with Godot LSP static typing validation
- `launch_project` — Run Godot game project in debug mode with captured engine logs and FPS metrics
- `import_asset` — Import textures, 3D glTF models, and audio files into `res://` project resources

## Client Configuration
```json
{
  "mcpServers": {
    "godot": {
      "command": "npx",
      "args": [
        "-y",
        "@godotengine/mcp-server",
        "--project-path=/path/to/godot_project"
      ]
    }
  }
}
```

## Security & Best Practices
- Local filesystem access within project root boundaries. Safe GDScript syntax linting.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
