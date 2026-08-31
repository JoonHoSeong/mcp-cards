# Hetzner Cloud Infrastructure MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | Hetzner Cloud Infrastructure MCP Server |
| **Category** | Cloud/Infrastructure |
| **Official** | ⭐ Official (Hetzner Cloud) |
| **Source** | https://github.com/hetznercloud/mcp-hcloud |
| **Transport** | stdio |
| **Install** | `npx -y @hetznercloud/mcp-server` |

## Tools & Capabilities
- `list_servers` — Query Hetzner cloud servers, IPv4/IPv6 addresses, datacenter regions, and status
- `create_server` — Provision cost-effective AMD/ARM cloud VPS with cloud-init bootstrap scripts
- `reboot_server` — Power cycle or soft reboot designated server instance
- `manage_floating_ips` — Assign and switch floating IP routes across cluster nodes

## Client Configuration
```json
{
  "mcpServers": {
    "hetzner": {
      "command": "npx",
      "args": [
        "-y",
        "@hetznercloud/mcp-server"
      ],
      "env": {
        "HCLOUD_TOKEN": "YOUR_HCLOUD_TOKEN"
      }
    }
  }
}
```

## Security & Best Practices
- Scoped Hetzner Cloud Read/Write API token. Prevent accidental deletion of production nodes.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
