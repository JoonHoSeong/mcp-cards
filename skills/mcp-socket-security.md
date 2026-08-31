# Socket.dev Supply Chain Security MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | Socket.dev Supply Chain Security MCP Server |
| **Category** | Security/Supply-Chain |
| **Official** | ⭐ Official (Socket.dev) |
| **Source** | https://github.com/SocketDev/socket-mcp |
| **Transport** | stdio |
| **Install** | `npx -y @socketdev/mcp-server` |

## Tools & Capabilities
- `audit_dependencies` — Scan `package.json` / `requirements.txt` for malicious packages, typosquatting, and install scripts
- `get_package_score` — Query Socket security, supply chain, maintenance, and quality scores (0-100)
- `inspect_package_alerts` — Retrieve CVEs, telemetry traps, hidden native binaries, and obfuscated code alerts
- `scan_diff_manifest` — Analyze git PR diffs for newly introduced insecure dependencies

## Client Configuration
```json
{
  "mcpServers": {
    "socket": {
      "command": "npx",
      "args": [
        "-y",
        "@socketdev/mcp-server"
      ],
      "env": {
        "SOCKET_SECURITY_API_KEY": "YOUR_SOCKET_KEY"
      }
    }
  }
}
```

## Security & Best Practices
- Socket API Key with read-only security assessment permissions.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
