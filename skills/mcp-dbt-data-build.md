# dbt Data Build Tool MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | dbt Data Build Tool MCP Server |
| **Category** | Data/Lineage |
| **Official** | ⭐ Official (dbt Labs) |
| **Source** | https://github.com/dbt-labs/mcp-dbt |
| **Transport** | stdio |
| **Install** | `npx -y @dbt-labs/mcp-server` |

## Tools & Capabilities
- `inspect_lineage` — Query DAG model dependencies, upstream sources, and downstream exposure marts
- `compile_model_sql` — Compile Jinja templated SQL models and inspect compiled SQL output
- `run_dbt_model` — Execute `dbt run --select <model>` with captured execution logs and status
- `run_tests` — Trigger schema tests and generic uniqueness/null validations

## Client Configuration
```json
{
  "mcpServers": {
    "dbt": {
      "command": "npx",
      "args": [
        "-y",
        "@dbt-labs/mcp-server"
      ],
      "env": {
        "DBT_PROJECT_DIR": "/path/to/dbt_project",
        "DBT_PROFILES_DIR": "~/.dbt"
      }
    }
  }
}
```

## Security & Best Practices
- Data warehouse credential isolation via standard `profiles.yml`. Dry-run SQL compilation supported.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
