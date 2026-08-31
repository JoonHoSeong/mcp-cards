# Lambda Labs GPU Cloud MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | Lambda Labs GPU Cloud MCP Server |
| **Category** | Cloud/GPU |
| **Official** | ⭐ Official (Lambda Labs) |
| **Source** | https://github.com/lambdalabs/mcp-lambda |
| **Transport** | stdio |
| **Install** | `npx -y @lambdalabs/mcp-server` |

## Tools & Capabilities
- `launch_instance` — Launch on-demand 8x H100 or 8x A100 GPU compute instances
- `list_instances` — Query active instance IPs, SSH keys, regions, and running status
- `terminate_instance` — Terminate cloud instances and release compute resources
- `list_instance_types` — Inspect available GPU instance types and hourly pricing

## Client Configuration
```json
{
  "mcpServers": {
    "lambda": {
      "command": "npx",
      "args": [
        "-y",
        "@lambdalabs/mcp-server"
      ],
      "env": {
        "LAMBDA_API_KEY": "secret_..."
      }
    }
  }
}
```

## Security & Best Practices
- Strict API key security. Terminate confirmation prompts for destructive actions.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
