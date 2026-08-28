# KakaoTalk Business MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | KakaoTalk Business MCP Server |
| **Category** | Communication/Korea |
| **Official** | ⭐ Official / Verified |
| **Source** | https://github.com/kakao/mcp-kakaotalk |
| **Transport** | stdio |
| **Install** | `npx -y @kakao/mcp-server` |

## Tools & Capabilities
- `send_alimtalk` — Send pre-approved KakaoTalk Alimtalk template message to customer phone number
- `send_friendtalk` — Send promotional Friendtalk message with image and interactive buttons
- `get_template_list` — Query approved Kakao business messaging template schemas and variables
- `get_message_status` — Inspect message delivery status, sent timestamps, and read receipts
- `send_memo_to_me` — Send private notification memo to authenticated Kakao user

## Client Configuration
```json
{
  "mcpServers": {
    "kakaotalk": {
      "command": "npx",
      "args": [
        "-y",
        "@kakao/mcp-server"
      ],
      "env": {
        "KAKAO_REST_API_KEY": "YOUR_KAKAO_REST_API_KEY",
        "KAKAO_SENDER_KEY": "YOUR_KAKAO_SENDER_KEY"
      }
    }
  }
}
```

## Security & Best Practices
- Enforce Alimtalk template variable validation. Do not leak customer phone numbers in logs.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
