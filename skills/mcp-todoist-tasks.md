# Todoist Task Management MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | Todoist Task Management MCP Server |
| **Category** | Productivity/Tasks |
| **Official** | 커뮤니티 검증 |
| **Source** | https://github.com/Doist/todoist-mcp |
| **Transport** | stdio |
| **Install** | `npx -y @doist/todoist-mcp` |

## Tools & Capabilities
- `get_tasks` — Query pending tasks by project, filter string (e.g. `today | overdue`), and priority
- `create_task` — Create new task with natural language due date parsing, labels, and subtasks
- `close_task` — Mark task as completed
- `list_projects` — List Todoist projects, color codes, and section groupings
- `update_task` — Modify task content, priority level (P1-P4), or reschedule due date

## Client Configuration
```json
{
  "mcpServers": {
    "todoist": {
      "command": "npx",
      "args": [
        "-y",
        "@doist/todoist-mcp"
      ],
      "env": {
        "TODOIST_API_TOKEN": "YOUR_TODOIST_TOKEN"
      }
    }
  }
}
```

## Security & Best Practices
- Scoped API token access. Prevent bulk task closure without user confirmation.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
