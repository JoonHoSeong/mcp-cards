# PagerDuty MCP Server

## Overview
| Field | Value |
|-------|-------|
| **Name** | pagerduty-mcp-server |
| **Category** | Incident Management |
| **Official** | Yes |
| **Source** | https://github.com/mcp/pagerduty/pagerduty-mcp-server |
| **Transport** | stdio |
| **Install** | `npx @pagerduty/mcp-server` |

## Description
Official PagerDuty MCP server for managing incidents, services, schedules, and on-call rotations. Provides AI-assisted incident lifecycle management including creation, acknowledgment, and resolution, along with service discovery and escalation policy queries.

## Key Tools
| Tool | Description |
|------|-------------|
| `list_incidents` | List incidents filtered by status, urgency, or service |
| `get_incident` | Get detailed incident information |
| `create_incident` | Create a new incident on a service |
| `acknowledge_incident` | Acknowledge an open incident |
| `resolve_incident` | Resolve an active incident |
| `list_services` | List all configured services |
| `list_schedules` | List on-call schedules |
| `list_oncalls` | Get current on-call personnel |

## Configuration
```json
{
  "mcpServers": {
    "pagerduty": {
      "command": "npx",
      "args": ["@pagerduty/mcp-server"],
      "env": {
        "PAGERDUTY_API_KEY": "your-api-key"
      }
    }
  }
}
```

## Use Cases
1. Automatically escalate video surveillance alerts as PagerDuty incidents
2. Query on-call schedules and acknowledge GPU infrastructure incidents via AI
3. Correlate inference service failures with incident timelines for root cause analysis

## NVIDIA Skill Combinations
| Purpose | NVIDIA Skill | Reason |
|---------|-------------|--------|
| Escalate video alerts | vss-manage-alerts | Escalate VSS video analytics alerts to PagerDuty for incident response |

## Prerequisites
- PagerDuty account with API access enabled
- `PAGERDUTY_API_KEY` — REST API key with full access or scoped OAuth token

## Security Notes
- Use scoped API keys restricted to specific services rather than account-wide keys
- Prefer read-only tokens for monitoring; use write tokens only for incident management

## References
- https://github.com/mcp/pagerduty/pagerduty-mcp-server
- https://developer.pagerduty.com/api-reference/
