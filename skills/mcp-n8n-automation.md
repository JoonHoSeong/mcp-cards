# n8n Workflow Automation MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | n8n Workflow Automation MCP Server |
| **Category** | Automation/Workflow |
| **Official** | ⭐ Official (n8n) |
| **Source** | https://github.com/n8n-io/mcp-n8n |
| **Transport** | stdio |
| **Install** | `npx -y @n8n/mcp-server` |

## Tools & Capabilities
- `list_workflows` — List active and inactive n8n workflows, triggers, and execution tags
- `execute_workflow` — Trigger specific workflow with dynamic JSON payload parameters
- `get_execution_status` — Inspect execution step outputs, error traces, and execution time
- `activate_workflow` — Enable or disable automated webhook/cron triggers for workflow
- `export_workflow_json` — Read complete node configuration and credential mappings

## Client Configuration
```json
{
  "mcpServers": {
    "n8n": {
      "command": "npx",
      "args": [
        "-y",
        "@n8n/mcp-server"
      ],
      "env": {
        "N8N_API_URL": "http://localhost:5678/api/v1",
        "N8N_API_KEY": "YOUR_N8N_API_KEY"
      }
    }
  }
}
```

## Security & Best Practices
- Restrict execution to approved webhook nodes. Enforce API key scope boundary.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
