# Cloudflare Zero Trust & Tunnels MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | Cloudflare Zero Trust & Tunnels MCP Server |
| **Category** | Security/Network |
| **Official** | ⭐ Official (Cloudflare) |
| **Source** | https://github.com/cloudflare/mcp-zero-trust |
| **Transport** | stdio |
| **Install** | `npx -y @cloudflare/mcp-zero-trust` |

## Tools & Capabilities
- `list_tunnels` — List Cloudflare cloudflared tunnels, connector status, and ingress rules
- `get_access_apps` — Inspect Access applications, session durations, and identity providers
- `manage_access_policies` — Audit allow/block rules, device posture checks, and email domain criteria
- `manage_dns_records` — Query and update CNAME/A records bound to Cloudflare Tunnels
- `get_gateway_logs` — Inspect Zero Trust Gateway DNS and HTTP filtering audit logs

## Client Configuration
```json
{
  "mcpServers": {
    "cloudflare-zero-trust": {
      "command": "npx",
      "args": [
        "-y",
        "@cloudflare/mcp-zero-trust"
      ],
      "env": {
        "CLOUDFLARE_API_TOKEN": "YOUR_CLOUDFLARE_API_TOKEN",
        "CLOUDFLARE_ACCOUNT_ID": "YOUR_ACCOUNT_ID"
      }
    }
  }
}
```

## Security & Best Practices
- Scoped Cloudflare API token with Zero Trust read/write permissions. Audit all policy changes.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
