# Fly.io MCP Server

## Overview
| Field | Value |
|-------|-------|
| **Name** | Fly.io MCP |
| **Category** | Edge Deployment |
| **Official** | Yes (Fly.io) |
| **Source** | https://github.com/superfly/flymcp |
| **Transport** | stdio |
| **Install** | `npx @flymcp/server` |

## Description
The Fly.io MCP Server enables AI agents to manage applications, machines, and infrastructure on Fly.io's global edge platform. It provides control over app deployments, machine lifecycle, volume management, and IP allocation — allowing agents to deploy and scale containerized workloads close to users across 30+ regions worldwide.

## Key Tools
| Tool | Description |
|------|-------------|
| `list_apps` | List all applications in the organization |
| `get_app_status` | Get the current status of an application |
| `list_machines` | List machines running for an app |
| `create_machine` | Create a new Fly Machine with specified config |
| `start_machine` | Start a stopped machine |
| `stop_machine` | Stop a running machine |
| `get_logs` | Retrieve application logs |
| `list_volumes` | List persistent volumes |
| `allocate_ip` | Allocate a dedicated IP address |

## Configuration
```json
{
  "mcpServers": {
    "flyio": {
      "command": "npx",
      "args": ["@flymcp/server"],
      "env": {
        "FLY_API_TOKEN": "your-fly-api-token"
      }
    }
  }
}
```

## Use Cases
1. Deploy GPU-accelerated inference containers to edge regions near users
2. Auto-scale machine count based on inference request volume
3. Manage multi-region AI service deployments with persistent volumes

## NVIDIA Skill Combinations
| Purpose | NVIDIA Skill | Reason |
|---------|-------------|--------|
| Edge inference | jetson-llm-serve | Deploy lightweight LLM serving containers to edge locations |
| Regional scaling | jetson-llm-serve | Scale inference machines across regions based on demand |
| Low-latency serving | jetson-llm-serve | Minimize inference latency by running models near users |

## Prerequisites
- Fly.io account with organization created
- `FLY_API_TOKEN` personal access token
- flyctl CLI installed for initial setup

## Security Notes
- Use organization-scoped tokens for team deployments
- Enable private networking between machines for internal traffic
- Store secrets using Fly.io's built-in secrets management
- Restrict machine access with Fly.io's firewall rules

## References
- https://github.com/superfly/flymcp
- https://fly.io/docs/machines/
- https://fly.io/docs/reference/configuration/
