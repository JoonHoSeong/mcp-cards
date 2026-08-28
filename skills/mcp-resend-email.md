# Resend MCP Server

## Overview
| Field | Value |
|-------|-------|
| **Name** | Resend MCP |
| **Category** | Email |
| **Official** | Official |
| **Source** | https://github.com/resend/resend-mcp |
| **Transport** | stdio |
| **Install** | `npx -y @resend/mcp` |

## Description
Resend MCP server enables AI agents to send transactional and broadcast emails, manage contacts, and monitor email delivery. It provides a developer-friendly email API through the Model Context Protocol, supporting automated notifications, user communications, and marketing campaigns from AI-powered workflows.

## Key Tools
| Tool | Description |
|------|-------------|
| `send_email` | Send a transactional email with HTML or text content |
| `list_emails` | List sent emails with filtering and pagination |
| `get_email` | Get delivery status and details of a specific email |
| `list_contacts` | List contacts in an audience with metadata |
| `add_contact` | Add a new contact to an audience list |
| `list_domains` | List verified sending domains |
| `list_broadcasts` | List broadcast campaigns and their status |
| `send_broadcast` | Send a broadcast email to an audience segment |

## Configuration
```json
{
  "mcpServers": {
    "resend": {
      "command": "npx",
      "args": ["-y", "@resend/mcp"],
      "env": {
        "RESEND_API_KEY": "<your-api-key>"
      }
    }
  }
}
```

## Use Cases
1. Send automated deployment notifications to team members  2. Alert stakeholders when AI model training completes  3. Manage contact lists for project update broadcasts

## NVIDIA Skill Combinations
| Purpose | NVIDIA Skill | Reason |
|---------|-------------|--------|
| Agent notifications | aiq-deploy | Send email alerts when AIQ toolkit deployments complete or fail |
| Training completion alerts | nemotron-customize | Notify team when Nemotron model customization finishes |
| Pipeline failure alerts | mcore-create-issue | Email notifications when Megatron-Core CI pipelines fail |

## Prerequisites
- `RESEND_API_KEY` from Resend dashboard
- Verified sending domain configured in Resend

## Security Notes
- API key can send emails on behalf of your domain; protect it carefully
- Rate limits apply; implement backoff for bulk operations
- Avoid sending to unverified recipients to maintain sender reputation

## References
- https://github.com/resend/resend-mcp
- https://resend.com/docs
