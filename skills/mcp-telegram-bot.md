# Telegram Bot API MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | Telegram Bot API MCP Server |
| **Category** | Communication/Messaging |
| **Official** | 커뮤니티 검증 |
| **Source** | https://github.com/telegram/mcp-telegram-bot |
| **Transport** | stdio |
| **Install** | `npx -y @telegram/mcp-bot` |

## Tools & Capabilities
- `send_message` — Send markdown/HTML formatted message to chat ID or channel
- `get_updates` — Poll incoming bot messages, commands, and callback queries
- `send_photo_or_doc` — Dispatch file attachments, images, or audio clips
- `send_inline_keyboard` — Render interactive callback button menus for user interaction
- `pin_chat_message` — Pin important announcements or alert headers in group chat

## Client Configuration
```json
{
  "mcpServers": {
    "telegram": {
      "command": "npx",
      "args": [
        "-y",
        "@telegram/mcp-bot"
      ],
      "env": {
        "TELEGRAM_BOT_TOKEN": "YOUR_BOT_TOKEN"
      }
    }
  }
}
```

## Security & Best Practices
- Validate chat ID whitelist to prevent sending messages to unauthorized external recipients.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
