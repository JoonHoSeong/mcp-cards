# Railway App Platform MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | Railway App Platform MCP Server |
| **Category** | Cloud/PaaS |
| **Official** | ⭐ Official (Railway) |
| **Source** | https://github.com/railwayapp/mcp-railway |
| **Transport** | stdio |
| **Install** | `npx -y @railwayapp/mcp-server` |

## Tools & Capabilities
- `deploy_project` — Deploy Railway project environment from latest git commit
- `list_variables` — Read and update environment variables across production/staging environments
- `get_deployment_logs` — Retrieve build and runtime stdout/stderr log streams
- `manage_plugins` — Provision and connect managed Redis, Postgres, and MySQL plugins

## Client Configuration
```json
{
  "mcpServers": {
    "railway": {
      "command": "npx",
      "args": [
        "-y",
        "@railwayapp/mcp-server"
      ],
      "env": {
        "RAILWAY_API_TOKEN": "YOUR_RAILWAY_TOKEN"
      }
    }
  }
}
```

## Security & Best Practices
- Railway Project/Personal Token. Mask secret variables in log responses.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
