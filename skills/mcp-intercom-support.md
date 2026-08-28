# Intercom MCP Server

## Overview
| Field | Value |
|-------|-------|
| **Name** | Intercom MCP |
| **Category** | Customer Support |
| **Official** | Yes (remote authenticated) |
| **Source** | https://github.com/intercom/intercom-mcp-server |
| **Transport** | Remote (HTTP + SSE, authenticated) |
| **Install** | Remote server — no local install required |

## Description
The Intercom MCP server provides AI agents with direct access to Intercom's customer support platform. It enables reading and managing conversations, contacts, and help center articles, allowing agents to analyze support patterns, triage tickets, and assist with customer communication workflows.

## Key Tools
| Tool | Description |
|------|-------------|
| `list_conversations` | List recent conversations with filters |
| `get_conversation` | Get full conversation details and messages |
| `reply_to_conversation` | Send a reply to a conversation |
| `list_contacts` | List contacts with filtering options |
| `get_contact` | Get detailed contact information |
| `search_articles` | Search help center articles |
| `list_teams` | List available support teams |
| `assign_conversation` | Assign conversation to team or agent |

## Configuration
```json
{
  "mcpServers": {
    "intercom": {
      "url": "https://intercom-mcp-server.intercom.com/mcp",
      "headers": {
        "Authorization": "Bearer ${INTERCOM_TOKEN}"
      }
    }
  }
}
```

## Use Cases
1. Automated support ticket triage and routing based on content analysis
2. Customer sentiment analysis across conversation history
3. AI-assisted reply drafting using help center knowledge

## NVIDIA Skill Combinations
| Purpose | NVIDIA Skill | Reason |
|---------|-------------|--------|
| Analyze support patterns | aiq-research | Mine conversation data for recurring issues and resolution patterns |

## Prerequisites
- `INTERCOM_TOKEN` — Intercom API access token with appropriate scopes

## Security Notes
- Token grants access to customer PII; restrict scopes to minimum required permissions
- All conversation data should be treated as sensitive customer information

## References
- https://github.com/intercom/intercom-mcp-server
- https://developers.intercom.com/docs
