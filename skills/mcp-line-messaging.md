# LINE MCP Server

## Overview
| Field | Value |
|-------|-------|
| **Name** | LINE Bot MCP |
| **Category** | Messaging |
| **Official** | Yes |
| **Source** | https://github.com/line/line-bot-mcp-server |
| **Transport** | stdio |
| **Install** | `npx @line/line-bot-mcp-server` |

## Description
The LINE Bot MCP server enables AI agents to interact with the LINE messaging platform. It supports sending messages, managing rich menus, retrieving user profiles, and broadcasting to followers, making it ideal for building conversational AI experiences on LINE.

## Key Tools
| Tool | Description |
|------|-------------|
| `send_message` | Send a message to a specific user |
| `get_profile` | Get user profile information |
| `list_followers` | List bot followers |
| `send_rich_menu` | Send or set rich menu for users |
| `get_message_content` | Retrieve message content (images, audio) |
| `send_multicast` | Send message to multiple users |
| `get_group_members` | List members in a group chat |

## Configuration
```json
{
  "mcpServers": {
    "line": {
      "command": "npx",
      "args": ["@line/line-bot-mcp-server"],
      "env": {
        "LINE_CHANNEL_ACCESS_TOKEN": "${LINE_CHANNEL_ACCESS_TOKEN}"
      }
    }
  }
}
```

## Use Cases
1. Automated customer engagement and notifications via LINE
2. AI chatbot responses with rich media content
3. Group management and broadcast messaging campaigns

## NVIDIA Skill Combinations
| Purpose | NVIDIA Skill | Reason |
|---------|-------------|--------|
| Voice messages | nemotron-speech | Convert text responses to voice messages for LINE audio delivery |

## Prerequisites
- `LINE_CHANNEL_ACCESS_TOKEN` — LINE Messaging API channel access token

## Security Notes
- Channel access token provides full messaging capability; rotate regularly
- Message content may contain user-uploaded media; handle with care

## References
- https://github.com/line/line-bot-mcp-server
- https://developers.line.biz/en/docs/messaging-api/
