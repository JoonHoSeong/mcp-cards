# Airtable No-Code Database MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | Airtable No-Code Database MCP Server |
| **Category** | Productivity/Database |
| **Official** | ⭐ Official (Airtable - mcp.airtable.com) |
| **Source** | https://github.com/airtable/mcp-airtable |
| **Transport** | stdio |
| **Install** | `npx -y @airtable/mcp-server` |

## Tools & Capabilities
- `list_records` — Query records from table with formula filters, sorting, and view selections
- `create_records` — Insert batch records with typed cell values (single select, linked records)
- `update_records` — Modify existing record fields with partial update support
- `get_base_schema` — Inspect tables, fields, relationship types, and formula definitions
- `delete_records` — Remove records by ID list

## Client Configuration
```json
{
  "mcpServers": {
    "airtable": {
      "command": "npx",
      "args": [
        "-y",
        "@airtable/mcp-server"
      ],
      "env": {
        "AIRTABLE_API_KEY": "pat...",
        "AIRTABLE_BASE_ID": "app..."
      }
    }
  }
}
```

## Security & Best Practices
- Use Scoped Personal Access Tokens (PAT). Validate field type schemas before insertion.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
