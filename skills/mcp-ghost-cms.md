# Ghost CMS Publication MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | Ghost CMS Publication MCP Server |
| **Category** | CMS/Publishing |
| **Official** | ⭐ Official (Ghost Foundation) |
| **Source** | https://github.com/tryghost/mcp-ghost |
| **Transport** | stdio |
| **Install** | `npx -y @tryghost/mcp-ghost` |

## Tools & Capabilities
- `create_post` — Draft or publish newsletters and articles with Mobiledoc / Lexical formats
- `list_members` — Query subscriber email tiers, subscriptions, and open rates
- `manage_tags` — Create and assign editorial tags and navigation categories
- `upload_feature_image` — Upload optimized header images directly to Ghost CDN

## Client Configuration
```json
{
  "mcpServers": {
    "ghost": {
      "command": "npx",
      "args": [
        "-y",
        "@tryghost/mcp-ghost"
      ],
      "env": {
        "GHOST_ADMIN_API_URL": "https://your-blog.ghost.io",
        "GHOST_ADMIN_API_KEY": "YOUR_ADMIN_API_KEY"
      }
    }
  }
}
```

## Security & Best Practices
- Ghost Admin API Key with JWT authentication. Draft post safety by default.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
