# HubSpot MCP Server

## Overview
| Field | Value |
|-------|-------|
| **Name** | HubSpot MCP |
| **Category** | CRM |
| **Official** | Yes |
| **Source** | https://github.com/hubspot/hubspot-mcp-server |
| **Transport** | stdio |
| **Install** | `npx @hubspot/mcp-server` |

## Description
The HubSpot MCP server connects AI agents to HubSpot's CRM platform for managing contacts, deals, and companies. It enables automated customer research, pipeline management, and activity logging, allowing agents to support sales and marketing workflows.

## Key Tools
| Tool | Description |
|------|-------------|
| `list_contacts` | List CRM contacts with filters |
| `get_contact` | Get detailed contact record |
| `create_contact` | Create a new contact |
| `list_deals` | List deals in pipeline |
| `update_deal` | Update deal stage or properties |
| `list_companies` | List company records |
| `log_activity` | Log a call, email, or meeting |
| `search_contacts` | Search contacts by properties |
| `get_pipeline` | Get pipeline stages and configuration |

## Configuration
```json
{
  "mcpServers": {
    "hubspot": {
      "command": "npx",
      "args": ["@hubspot/mcp-server"],
      "env": {
        "HUBSPOT_ACCESS_TOKEN": "${HUBSPOT_ACCESS_TOKEN}"
      }
    }
  }
}
```

## Use Cases
1. AI-driven lead scoring and deal prioritization
2. Automated activity logging and contact enrichment
3. Sales pipeline analysis and forecasting

## NVIDIA Skill Combinations
| Purpose | NVIDIA Skill | Reason |
|---------|-------------|--------|
| Customer research | aiq-research | Deep research into customer profiles and deal history for strategic insights |

## Prerequisites
- `HUBSPOT_ACCESS_TOKEN` — HubSpot private app access token with CRM scopes

## Security Notes
- Access token exposes customer PII and sales data; use minimum required scopes
- Activity logging creates permanent records; validate before writing

## References
- https://github.com/hubspot/hubspot-mcp-server
- https://developers.hubspot.com/docs/api
