# Asana Project Management MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | Asana Project Management MCP Server |
| **Category** | Productivity/PM |
| **Official** | ⭐ Official (Asana) |
| **Source** | https://github.com/asana/mcp-asana |
| **Transport** | stdio |
| **Install** | `npx -y @asana/mcp-server` |

## Tools & Capabilities
- `search_tasks` — Search workspace tasks by project, assignee, completion state, and tags
- `create_task` — Create task within target project with due dates and custom field mappings
- `update_task` — Modify task fields, mark complete, or reassign ownership
- `list_projects` — Query active projects, portfolios, and workflow custom fields
- `add_comment` — Append status update or blocker note to task story stream

## Client Configuration
```json
{
  "mcpServers": {
    "asana": {
      "command": "npx",
      "args": [
        "-y",
        "@asana/mcp-server"
      ],
      "env": {
        "ASANA_ACCESS_TOKEN": "1/..."
      }
    }
  }
}
```

## Security & Best Practices
- Personal access token isolation. Prevent unintended task deletion through audit checks.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
