# Replicate Model Deployments MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | Replicate Model Deployments MCP Server |
| **Category** | AI/Deployment |
| **Official** | ⭐ Official (Replicate) |
| **Source** | https://github.com/replicate/mcp-replicate |
| **Transport** | stdio |
| **Install** | `npx -y @replicate/mcp-deployments` |

## Tools & Capabilities
- `create_deployment` — Create dedicated private model deployment on specific GPU hardware tier
- `update_deployment` — Update minimum/maximum instance scaling and model version
- `list_deployments` — Inspect active deployments, traffic routing, and queue depths
- `get_hardware_skus` — List available hardware SKUs (CPU, Nvidia T4, A100 80GB, H100)

## Client Configuration
```json
{
  "mcpServers": {
    "replicate-deployments": {
      "command": "npx",
      "args": [
        "-y",
        "@replicate/mcp-deployments"
      ],
      "env": {
        "REPLICATE_API_TOKEN": "r8_..."
      }
    }
  }
}
```

## Security & Best Practices
- Enforce minimum replica scaling rules to control idle resource consumption.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
