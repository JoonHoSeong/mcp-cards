# Databricks Lakehouse MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | Databricks Lakehouse MCP Server |
| **Category** | Data/Analytics |
| **Official** | ⭐ Official (Databricks) |
| **Source** | https://github.com/databricks/mcp-databricks |
| **Transport** | stdio |
| **Install** | `npx -y @databricks/mcp-server` |

## Tools & Capabilities
- `execute_sql` — Run queries against Databricks SQL Warehouses (Unity Catalog)
- `list_catalogs_and_schemas` — Inspect data tables, volumes, and column types in Unity Catalog
- `get_job_runs` — Monitor Databricks Workflow jobs and Spark cluster execution status
- `list_mlflow_experiments` — Query registered models and training run parameters in MLflow
- `manage_vector_search_indexes` — Query and sync Databricks Vector Search endpoints

## Client Configuration
```json
{
  "mcpServers": {
    "databricks": {
      "command": "npx",
      "args": [
        "-y",
        "@databricks/mcp-server"
      ],
      "env": {
        "DATABRICKS_HOST": "https://your-instance.cloud.databricks.com",
        "DATABRICKS_TOKEN": "dapi..."
      }
    }
  }
}
```

## Security & Best Practices
- Enforce Unity Catalog data access policies. Restrict SQL execution timeout to avoid cluster overruns.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
