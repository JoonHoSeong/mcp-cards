# X (Twitter) Developer API MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | X (Twitter) Developer API MCP Server |
| **Category** | Social/Media |
| **Official** | ⭐ Official (X API v2) |
| **Source** | https://github.com/xdev/mcp-twitter |
| **Transport** | stdio |
| **Install** | `npx -y @xdev/mcp-server` |

## Tools & Capabilities
- `search_recent_posts` — Search posts by keywords, hashtags, author, and language
- `post_tweet` — Publish new post thread with optional media attachment IDs
- `get_user_timeline` — Retrieve public timeline and engagement metrics for user handle
- `get_post_metrics` — Inspect impressions, likes, retweets, and bookmark analytics
- `manage_bookmarks` — Save important posts to authenticated user's private bookmarks

## Client Configuration
```json
{
  "mcpServers": {
    "twitter": {
      "command": "npx",
      "args": [
        "-y",
        "@xdev/mcp-server"
      ],
      "env": {
        "X_API_BEARER_TOKEN": "YOUR_BEARER_TOKEN",
        "X_API_KEY": "YOUR_API_KEY",
        "X_API_SECRET": "YOUR_API_SECRET"
      }
    }
  }
}
```

## Security & Best Practices
- Post creation requires user confirmation. Respect rate limits (15-minute window quotas).

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
