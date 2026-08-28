# Google BigQuery MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | Google BigQuery MCP Server |
| **Category** | Database/Warehouse |
| **Official** | ⭐ Official (Google Cloud) |
| **Source** | https://github.com/google-cloud/mcp-bigquery |
| **Transport** | stdio |
| **Install** | `npx -y @google-cloud/mcp-bigquery` |

## Tools & Capabilities
- `list_datasets` — List BigQuery datasets, locations, and access controls
- `get_table_schema` — Inspect columns, partition specs, and clustering fields
- `run_query` — Execute standard SQL query with dry-run byte estimation and limits
- `estimate_query_cost` — Calculate processed bytes and estimated cost prior to execution
- `export_table_data` — Export query results to GCS or formatted CSV/JSON

## Client Configuration
```json
{
  "mcpServers": {
    "bigquery": {
      "command": "npx",
      "args": [
        "-y",
        "@google-cloud/mcp-bigquery"
      ],
      "env": {
        "GOOGLE_APPLICATION_CREDENTIALS": "/path/to/sa.json",
        "BIGQUERY_PROJECT_ID": "your-project-id"
      }
    }
  }
}
```

## Security & Best Practices
- Always enforce dry-run validation. Restrict maximum billed bytes per query to prevent unexpected charges.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
