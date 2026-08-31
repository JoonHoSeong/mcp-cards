# RunPod GPU Cloud MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | RunPod GPU Cloud MCP Server |
| **Category** | Cloud/GPU |
| **Official** | ⭐ Official (RunPod) |
| **Source** | https://github.com/runpod/mcp-runpod |
| **Transport** | stdio |
| **Install** | `npx -y @runpod/mcp-server` |

## Tools & Capabilities
- `create_pod` — Spin up GPU compute pods with custom Docker templates and disk size
- `stop_pod` — Terminate or stop idle pods to prevent unnecessary billing
- `list_gpu_availability` — Query real-time GPU stock (H100 SXM, RTX 4090, A6000) and pricing
- `manage_serverless_endpoints` — Deploy auto-scaling serverless AI worker endpoints

## Client Configuration
```json
{
  "mcpServers": {
    "runpod": {
      "command": "npx",
      "args": [
        "-y",
        "@runpod/mcp-server"
      ],
      "env": {
        "RUNPOD_API_KEY": "rpa_..."
      }
    }
  }
}
```

## Security & Best Practices
- Require confirmation for pod creation to prevent accidental cloud spend.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
