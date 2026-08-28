# Raycast MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | Raycast MCP Server |
| **Category** | Productivity/Desktop |
| **Official** | ⭐ Official (Raycast) |
| **Source** | https://github.com/raycast/mcp-raycast |
| **Transport** | stdio |
| **Install** | `npx -y @raycast/mcp-server` |

## Tools & Capabilities
- `execute_command` — Trigger Raycast commands and script extensions
- `search_clipboard` — Query local clipboard history with semantic filtering
- `manage_snippets` — Insert, update, and search text expansion snippets
- `list_windows` — Retrieve active macOS application windows and focus state
- `open_quicklink` — Launch configured bookmarks, deep links, and system actions

## Client Configuration
```json
{
  "mcpServers": {
    "raycast": {
      "command": "npx",
      "args": [
        "-y",
        "@raycast/mcp-server"
      ]
    }
  }
}
```

## Security & Best Practices
- macOS local IPC only. Clipboard access requires explicit local user permission.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
