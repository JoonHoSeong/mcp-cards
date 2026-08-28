# Bitwarden / Vaultwarden Secrets MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | Bitwarden / Vaultwarden Secrets MCP Server |
| **Category** | Security/Secrets |
| **Official** | 커뮤니티 검증 |
| **Source** | https://github.com/bitwarden/mcp-bitwarden |
| **Transport** | stdio |
| **Install** | `npx -y @bitwarden/mcp-server` |

## Tools & Capabilities
- `search_vault` — Search items by name, domain, or folder in unlocked vault
- `get_password` — Retrieve password or TOTP authentication code for specific item ID
- `get_secure_note` — Read encrypted secure note text and attachment metadata
- `generate_password` — Generate cryptographically secure random password by policy
- `sync_vault` — Trigger immediate vault synchronization with Bitwarden server

## Client Configuration
```json
{
  "mcpServers": {
    "bitwarden": {
      "command": "npx",
      "args": [
        "-y",
        "@bitwarden/mcp-server"
      ],
      "env": {
        "BW_SESSION": "YOUR_UNLOCKED_BW_SESSION_KEY"
      }
    }
  }
}
```

## Security & Best Practices
- Never store master password. Master vault session token must remain memory-only and expire automatically.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
