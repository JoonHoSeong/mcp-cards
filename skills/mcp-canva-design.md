# Canva MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | Canva MCP Server |
| **Category** | Frontend/Design |
| **Official** | ⭐ Official (Canva) |
| **Source** | https://github.com/canva/mcp-canva |
| **Transport** | stdio |
| **Install** | `npx -y @canva/mcp-server` |

## Tools & Capabilities
- `search_templates` — Search Canva design templates by query, tags, and dimensions
- `create_design` — Initialize a new design project with preset dimensions
- `export_design` — Export design to PNG, JPEG, PDF, or MP4 formats
- `list_brand_assets` — Retrieve logos, color palettes, and typography assets
- `generate_content` — Insert automated text, images, and visual elements

## Client Configuration
```json
{
  "mcpServers": {
    "canva": {
      "command": "npx",
      "args": [
        "-y",
        "@canva/mcp-server"
      ],
      "env": {
        "CANVA_API_KEY": "YOUR_CANVA_API_KEY"
      }
    }
  }
}
```

## Security & Best Practices
- Enforce read-only scopes for template search. Require explicit user approval before triggering asset export or modifying brand kits.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
