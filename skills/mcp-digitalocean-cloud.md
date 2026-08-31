# DigitalOcean Cloud MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | DigitalOcean Cloud MCP Server |
| **Category** | Cloud/Infrastructure |
| **Official** | ⭐ Official (DigitalOcean) |
| **Source** | https://github.com/digitalocean/mcp-digitalocean |
| **Transport** | stdio |
| **Install** | `npx -y @digitalocean/mcp-server` |

## Tools & Capabilities
- `list_droplets` — Query Droplet virtual machines, regions, IP addresses, and status
- `manage_apps` — Deploy, restart, and inspect DigitalOcean App Platform services
- `list_kubernetes_clusters` — Inspect DOKS clusters, node pools, and kubeconfig credentials
- `manage_databases` — Query managed PostgreSQL, MySQL, Redis, and OpenSearch clusters
- `get_account_balance` — Inspect billing usage, credit balance, and invoices

## Client Configuration
```json
{
  "mcpServers": {
    "digitalocean": {
      "command": "npx",
      "args": [
        "-y",
        "@digitalocean/mcp-server"
      ],
      "env": {
        "DIGITALOCEAN_TOKEN": "dop_v1_..."
      }
    }
  }
}
```

## Security & Best Practices
- Scoped Personal Access Token. Restrict droplet deletion tools with user confirmation.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
