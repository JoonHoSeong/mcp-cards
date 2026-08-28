# Monday.com Work OS MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | Monday.com Work OS MCP Server |
| **Category** | Productivity/PM |
| **Official** | ⭐ Official (Monday.com) |
| **Source** | https://github.com/mondaycom/mcp-monday |
| **Transport** | stdio |
| **Install** | `npx -y @mondaycom/mcp` |

## Tools & Capabilities
- `query_boards` — List workspace boards, groups, columns, and permission models
- `get_items` — Query board items with column values (status, timeline, numbers, people)
- `create_item` — Add new item to board with structured column payload JSON
- `change_column_value` — Mutate specific column state (e.g., mark Status as Done)
- `post_update` — Add activity update and threaded message to item

## Client Configuration
```json
{
  "mcpServers": {
    "monday": {
      "command": "npx",
      "args": [
        "-y",
        "@mondaycom/mcp"
      ],
      "env": {
        "MONDAY_API_TOKEN": "YOUR_MONDAY_TOKEN"
      }
    }
  }
}
```

## Security & Best Practices
- Use GraphQL API token with board-level read/write permissions.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
