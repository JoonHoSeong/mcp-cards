# Zendesk Customer Support MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | Zendesk Customer Support MCP Server |
| **Category** | Enterprise/Support |
| **Official** | ⭐ Official (Zendesk) |
| **Source** | https://github.com/zendesk/mcp-zendesk |
| **Transport** | stdio |
| **Install** | `npx -y @zendesk/mcp-server` |

## Tools & Capabilities
- `search_tickets` — Query support tickets by status, priority, tags, requester, and assignee
- `create_ticket` — Create support ticket with subject, body description, and custom fields
- `update_ticket_status` — Resolve, close, or reassign tickets with internal private notes
- `apply_macro` — Execute pre-approved support macro actions and canned responses
- `search_help_center` — Query Zendesk Guide knowledge base articles for customer self-service

## Client Configuration
```json
{
  "mcpServers": {
    "zendesk": {
      "command": "npx",
      "args": [
        "-y",
        "@zendesk/mcp-server"
      ],
      "env": {
        "ZENDESK_SUBDOMAIN": "your-subdomain",
        "ZENDESK_EMAIL": "agent@company.com",
        "ZENDESK_API_TOKEN": "YOUR_ZENDESK_API_TOKEN"
      }
    }
  }
}
```

## Security & Best Practices
- Enforce agent role permissions. Public customer replies require confirmation.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
