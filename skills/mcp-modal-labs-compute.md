# Modal Labs Serverless Cloud MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | Modal Labs Serverless Cloud MCP Server |
| **Category** | Cloud/Serverless |
| **Official** | ⭐ Official (Modal Labs) |
| **Source** | https://github.com/modal-labs/mcp-modal |
| **Transport** | stdio |
| **Install** | `npx -y @modal-labs/mcp-server` |

## Tools & Capabilities
- `deploy_modal_app` — Deploy serverless Python apps with GPU attachments (H100, A100, L40S)
- `run_function` — Trigger remote distributed Python functions and retrieve return results
- `inspect_app_logs` — Stream real-time stdout/stderr logs from running container instances
- `manage_volumes` — Inspect and synchronize persistent network storage volumes

## Client Configuration
```json
{
  "mcpServers": {
    "modal": {
      "command": "npx",
      "args": [
        "-y",
        "@modal-labs/mcp-server"
      ],
      "env": {
        "MODAL_TOKEN_ID": "ak-...",
        "MODAL_TOKEN_SECRET": "as-..."
      }
    }
  }
}
```

## Security & Best Practices
- Modal token pair authentication. Enforce container timeout and GPU memory limits.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
