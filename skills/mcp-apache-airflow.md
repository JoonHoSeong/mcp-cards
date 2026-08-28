# Apache Airflow MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | Apache Airflow MCP Server |
| **Category** | Data/Orchestration |
| **Official** | 커뮤니티 검증 |
| **Source** | https://github.com/apache/airflow-mcp |
| **Transport** | stdio |
| **Install** | `npx -y airflow-mcp-server` |

## Tools & Capabilities
- `list_dags` — List active Airflow DAGs, schedules, and pause status
- `trigger_dag_run` — Trigger manual DAG run with JSON configuration parameters
- `get_dag_run_status` — Inspect task instance execution states (success, running, failed)
- `get_task_logs` — Retrieve stdout/stderr logs for failed task attempts
- `clear_failed_tasks` — Retry failed task instances in active DAG runs

## Client Configuration
```json
{
  "mcpServers": {
    "airflow": {
      "command": "npx",
      "args": [
        "-y",
        "airflow-mcp-server"
      ],
      "env": {
        "AIRFLOW_BASE_URL": "http://localhost:8080",
        "AIRFLOW_USERNAME": "admin",
        "AIRFLOW_PASSWORD": "admin"
      }
    }
  }
}
```

## Security & Best Practices
- Require admin verification before clearing tasks or triggering heavy backfill DAG runs.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
