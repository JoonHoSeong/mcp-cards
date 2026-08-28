# Tailscale Mesh VPN MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | Tailscale Mesh VPN MCP Server |
| **Category** | Network/VPN |
| **Official** | ⭐ Official (Tailscale) |
| **Source** | https://github.com/tailscale/mcp-tailscale |
| **Transport** | stdio |
| **Install** | `npx -y @tailscale/mcp-server` |

## Tools & Capabilities
- `list_devices` — List tailnet connected nodes, OS, tailscale IPs, and online status
- `get_device_routes` — Inspect advertised and approved subnet routes and exit nodes
- `check_acl_policy` — Validate HuJSON access control rules and tag permissions
- `generate_auth_key` — Create ephemeral or reusable pre-authentication keys for new servers
- `inspect_tailscale_ssh` — Check SSH reachability and user policy constraints

## Client Configuration
```json
{
  "mcpServers": {
    "tailscale": {
      "command": "npx",
      "args": [
        "-y",
        "@tailscale/mcp-server"
      ],
      "env": {
        "TAILSCALE_API_KEY": "tskey-api-...",
        "TAILSCALE_TAILNET": "your-tailnet.ts.net"
      }
    }
  }
}
```

## Security & Best Practices
- Protect Tailscale API keys. Auth key generation is restricted to ephemeral node tags.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
