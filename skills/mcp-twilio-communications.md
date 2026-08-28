# Twilio MCP Server

## Overview
| Field | Value |
|-------|-------|
| **Name** | Twilio MCP |
| **Category** | Communications |
| **Official** | Official |
| **Source** | https://github.com/twilio-labs/mcp |
| **Transport** | stdio |
| **Install** | `npx -y @twilio-labs/mcp` |

## Description
Twilio MCP server provides AI agents with access to over 1800 API endpoints across 30+ Twilio products including SMS, voice, video, and verification services. It enables automated communications workflows, allowing agents to send notifications, make calls, verify users, and manage messaging infrastructure through the Model Context Protocol.

## Key Tools
| Tool | Description |
|------|-------------|
| `send_sms` | Send an SMS message to a phone number |
| `make_call` | Initiate a voice call with TwiML instructions |
| `list_messages` | List sent and received messages with filters |
| `get_account_info` | Get Twilio account details and usage |
| `search_docs` | Search Twilio documentation and guides |
| `get_api_spec` | Get API specification for a Twilio product |
| `create_verify_service` | Create a phone verification service |

## Configuration
```json
{
  "mcpServers": {
    "twilio": {
      "command": "npx",
      "args": ["-y", "@twilio-labs/mcp"],
      "env": {
        "TWILIO_ACCOUNT_SID": "<your-account-sid>",
        "TWILIO_AUTH_TOKEN": "<your-auth-token>"
      }
    }
  }
}
```

## Use Cases
1. Send SMS/voice alerts when AI model deployments complete or fail  2. Implement phone verification for AI application users  3. Build voice-activated AI agent interfaces with Twilio Voice

## NVIDIA Skill Combinations
| Purpose | NVIDIA Skill | Reason |
|---------|-------------|--------|
| Agent SMS/voice alerts | aiq-deploy | Notify operators via SMS/voice when AIQ deployments need attention |
| Video analytics alerts | vss-manage-alerts | Send SMS alerts when video analytics detects critical events |
| Training notifications | nemotron-customize | Voice call notifications for long-running model training completion |

## Prerequisites
- `TWILIO_ACCOUNT_SID` from Twilio console
- `TWILIO_AUTH_TOKEN` from Twilio console
- Active Twilio phone number for sending

## Security Notes
- Auth token provides full account access; rotate regularly and never commit to code
- SMS/voice costs money; implement rate limiting and budget alerts
- Phone numbers in logs constitute PII; handle per data protection regulations

## References
- https://github.com/twilio-labs/mcp
- https://www.twilio.com/docs
