# ClickUp Task Management MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | ClickUp Task Management MCP Server |
| **Category** | Productivity/PM |
| **Official** | ⭐ Official (ClickUp) |
| **Source** | https://github.com/clickup/mcp-clickup |
| **Transport** | stdio |
| **Install** | `npx -y @clickup/mcp-server` |

## Tools & Capabilities
- `get_tasks` — Query tasks across spaces, folders, and lists with status/assignee filters
- `create_task` — Create new task with markdown description, priority, due date, and tags
- `update_task` — Update task status, custom field values, time estimates, and assignees
- `add_task_comment` — Post discussion updates or resolution notes to task thread
- `list_spaces_and_lists` — Traverse workspace hierarchy to identify target list IDs

## Client Configuration
```json
{
  "mcpServers": {
    "clickup": {
      "command": "npx",
      "args": [
        "-y",
        "@clickup/mcp-server"
      ],
      "env": {
        "CLICKUP_API_TOKEN": "pk_...",
        "CLICKUP_TEAM_ID": "123456"
      }
    }
  }
}
```

## Security & Best Practices
- Enforce team workspace boundaries. Mask internal email addresses in task payloads.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
