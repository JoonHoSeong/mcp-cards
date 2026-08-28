# Reddit Community API MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | Reddit Community API MCP Server |
| **Category** | Social/Community |
| **Official** | 커뮤니티 검증 |
| **Source** | https://github.com/reddit/mcp-reddit |
| **Transport** | stdio |
| **Install** | `npx -y @reddit/mcp-server` |

## Tools & Capabilities
- `search_subreddit` — Search posts across specific subreddit with sort by hot/new/top
- `get_post_thread` — Retrieve post text, upvote ratios, and nested comment tree
- `get_user_submissions` — Fetch submission history and karma breakdown for username
- `submit_post` — Create text or link submission to designated subreddit
- `submit_comment` — Reply to specific comment or thread ID

## Client Configuration
```json
{
  "mcpServers": {
    "reddit": {
      "command": "npx",
      "args": [
        "-y",
        "@reddit/mcp-server"
      ],
      "env": {
        "REDDIT_CLIENT_ID": "YOUR_CLIENT_ID",
        "REDDIT_CLIENT_SECRET": "YOUR_CLIENT_SECRET",
        "REDDIT_USER_AGENT": "mcp-bot:v1.0.0"
      }
    }
  }
}
```

## Security & Best Practices
- Strict adherence to Reddit Data API terms. Posting actions require manual user confirmation.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
