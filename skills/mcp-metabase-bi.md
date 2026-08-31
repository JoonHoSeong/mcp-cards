# Metabase Business Intelligence MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | Metabase Business Intelligence MCP Server |
| **Category** | Data/BI |
| **Official** | ⭐ Official (Metabase) |
| **Source** | https://github.com/metabase/mcp-metabase |
| **Transport** | stdio |
| **Install** | `npx -y @metabase/mcp-server` |

## Tools & Capabilities
- `query_question` — Execute saved analytical questions or native SQL queries and return structured results
- `list_dashboards` — Query organization dashboards, visual cards, and parameter filters
- `inspect_database_metadata` — Inspect table schemas, foreign key relationships, and metric definitions
- `create_card` — Programmatically scaffold new analytical question card

## Client Configuration
```json
{
  "mcpServers": {
    "metabase": {
      "command": "npx",
      "args": [
        "-y",
        "@metabase/mcp-server"
      ],
      "env": {
        "METABASE_URL": "https://metabase.example.com",
        "METABASE_API_KEY": "mb_..."
      }
    }
  }
}
```

## Security & Best Practices
- Metabase API Key with group-level data access permissions. Read-only query enforcement.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
