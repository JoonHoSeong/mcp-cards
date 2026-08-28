# SurrealDB Multi-Model Database MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | SurrealDB Multi-Model Database MCP Server |
| **Category** | Database/Multi-Model |
| **Official** | ⭐ Official (SurrealDB) |
| **Source** | https://github.com/surrealdb/mcp-surrealdb |
| **Transport** | stdio |
| **Install** | `npx -y @surrealdb/mcp-server` |

## Tools & Capabilities
- `execute_surql` — Execute SurrealQL queries across document, graph, and relational paradigms
- `get_schema_info` — Inspect defined tables, record links, permissions, and live events
- `create_record` — Insert complex nested JSON records with record link relations
- `traverse_graph` — Query graph relationships using `->` and `<-` syntax
- `manage_live_queries` — Subscribe and stream real-time change events

## Client Configuration
```json
{
  "mcpServers": {
    "surrealdb": {
      "command": "npx",
      "args": [
        "-y",
        "@surrealdb/mcp-server"
      ],
      "env": {
        "SURREAL_URL": "http://127.0.0.1:8000",
        "SURREAL_NS": "test",
        "SURREAL_DB": "test",
        "SURREAL_USER": "root",
        "SURREAL_PASS": "root"
      }
    }
  }
}
```

## Security & Best Practices
- Scoped namespace and database isolation. Parameterize SurrealQL variables.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
