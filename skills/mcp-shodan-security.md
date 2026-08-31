# Shodan Cybersecurity Reconnaissance MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | Shodan Cybersecurity Reconnaissance MCP Server |
| **Category** | Security/Recon |
| **Official** | ⭐ Official (Shodan) |
| **Source** | https://github.com/shodan/mcp-shodan |
| **Transport** | stdio |
| **Install** | `npx -y @shodan/mcp-server` |

## Tools & Capabilities
- `host_search` — Query Shodan database for IP address details, open ports, banners, and vulnerabilities (CVEs)
- `search_query` — Search internet-connected infrastructure using Shodan search filters (e.g. `product:nginx`)
- `dns_lookup` — Resolve domain hostnames and discover subdomains and IP history
- `scan_ip` — Request real-time port scan for authorized IP address range
- `get_account_info` — Check API query credits and scan credit balance

## Client Configuration
```json
{
  "mcpServers": {
    "shodan": {
      "command": "npx",
      "args": [
        "-y",
        "@shodan/mcp-server"
      ],
      "env": {
        "SHODAN_API_KEY": "YOUR_SHODAN_API_KEY"
      }
    }
  }
}
```

## Security & Best Practices
- Authorized security assessment and reconnaissance only. Guard against unauthorized scanning.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
