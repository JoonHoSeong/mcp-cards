# Slack MCP Server

## Overview
| Field | Value |
|-------|-------|
| **Name** | slack-mcp-server |
| **Category** | Communication |
| **Official** | Community (best implementation) |
| **Source** | https://github.com/korotovsky/slack-mcp-server |
| **Transport** | stdio + SSE |
| **Install** | `npx slack-mcp-server` |

## Description
Community Slack MCP server providing comprehensive messaging capabilities including channel management, message posting, search, and file uploads. Supports both stdio and SSE transports for flexible integration with AI workflows that need to communicate alerts, reports, and notifications through Slack workspaces.

## Key Tools
| Tool | Description |
|------|-------------|
| `list_channels` | List channels in the workspace |
| `read_channel_history` | Read recent messages from a channel |
| `post_message` | Post a message to a channel or thread |
| `search_messages` | Search messages across the workspace |
| `get_thread` | Get all replies in a message thread |
| `list_users` | List workspace members |
| `upload_file` | Upload a file to a channel |

## Configuration
```json
{
  "mcpServers": {
    "slack": {
      "command": "npx",
      "args": ["slack-mcp-server"],
      "env": {
        "SLACK_BOT_TOKEN": "xoxb-your-bot-token"
      }
    }
  }
}
```

## Use Cases
1. Post video surveillance alerts from VSS pipelines to dedicated Slack channels
2. Search historical messages for incident context during GPU cluster troubleshooting
3. Upload inference result summaries and model performance reports to team channels

## NVIDIA Skill Combinations
| Purpose | NVIDIA Skill | Reason |
|---------|-------------|--------|
| Alert to Slack | vss-manage-alerts | Route VSS video analytics alerts as Slack notifications |

## Prerequisites
- Slack workspace with a bot app installed
- `SLACK_BOT_TOKEN` — Bot user OAuth token (xoxb-) with appropriate scopes

## Security Notes
- Request minimal bot scopes (channels:read, chat:write, files:write) per use case
- Do not grant admin scopes unless workspace management is explicitly required

## References
- https://github.com/korotovsky/slack-mcp-server
- https://api.slack.com/methods
