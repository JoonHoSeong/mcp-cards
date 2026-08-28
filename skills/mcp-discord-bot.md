# Discord Bot & Community MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | Discord Bot & Community MCP Server |
| **Category** | Communication/Community |
| **Official** | 커뮤니티 검증 |
| **Source** | https://github.com/discord/mcp-discord |
| **Transport** | stdio |
| **Install** | `npx -y @discord/mcp-server` |

## Tools & Capabilities
- `send_message` — Post formatted text, embeds, and action buttons to channel
- `read_messages` — Fetch recent message history and thread replies with author info
- `list_channels` — Inspect server guild channels, categories, and permission overwrites
- `manage_roles` — Assign or remove user roles based on community actions
- `create_thread` — Spawn sub-discussion thread on specific message

## Client Configuration
```json
{
  "mcpServers": {
    "discord": {
      "command": "npx",
      "args": [
        "-y",
        "@discord/mcp-server"
      ],
      "env": {
        "DISCORD_BOT_TOKEN": "YOUR_DISCORD_BOT_TOKEN"
      }
    }
  }
}
```

## Security & Best Practices
- Enforce Discord Bot permissions allowlist. Block message deletion outside dedicated bot channels.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
