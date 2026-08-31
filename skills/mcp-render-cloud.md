# Render Cloud Platform MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | Render Cloud Platform MCP Server |
| **Category** | Cloud/PaaS |
| **Official** | ⭐ Official (Render) |
| **Source** | https://github.com/render-oss/mcp-render |
| **Transport** | stdio |
| **Install** | `npx -y @render-oss/mcp-server` |

## Tools & Capabilities
- `trigger_deploy` — Trigger clear-build-cache deployments for web services and background workers
- `list_services` — Inspect services, domains, environment configurations, and health status
- `get_service_logs` — Stream deployment and runtime application logs
- `manage_postgres` — Inspect managed PostgreSQL database connection strings and scaling

## Client Configuration
```json
{
  "mcpServers": {
    "render": {
      "command": "npx",
      "args": [
        "-y",
        "@render-oss/mcp-server"
      ],
      "env": {
        "RENDER_API_KEY": "rnd_..."
      }
    }
  }
}
```

## Security & Best Practices
- Scoped API token. Confirmation required for service suspension and deletions.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
