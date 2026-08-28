# Sentry MCP Server

## Overview
| Field | Value |
|-------|-------|
| **Name** | Sentry MCP |
| **Category** | Error Tracking |
| **Official** | Reference |
| **Source** | https://github.com/modelcontextprotocol/servers/tree/main/src/sentry |
| **Transport** | stdio |
| **Install** | `npx -y @modelcontextprotocol/server-sentry` |

## Description
Sentry MCP server enables AI agents to access error tracking data, investigate issues, review event details, and manage alert configurations. It provides visibility into application health by exposing crash reports, performance issues, and release tracking through the Model Context Protocol for automated incident response.

## Key Tools
| Tool | Description |
|------|-------------|
| `list_issues` | List error issues with frequency and status filters |
| `get_issue` | Get detailed issue information with stack trace and context |
| `list_events` | List error events for a specific issue |
| `get_event` | Get full event details including breadcrumbs and tags |
| `list_projects` | List monitored projects and their platforms |
| `get_release` | Get release information with deploy and commit data |
| `resolve_issue` | Mark an issue as resolved with resolution details |
| `list_alerts` | List configured alert rules and their trigger status |

## Configuration
```json
{
  "mcpServers": {
    "sentry": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-sentry"],
      "env": {
        "SENTRY_AUTH_TOKEN": "<your-auth-token>",
        "SENTRY_ORG": "<your-organization-slug>"
      }
    }
  }
}
```

## Use Cases
1. Automated error triage and assignment based on stack trace analysis  2. Correlate deployment events with error rate spikes  3. Auto-resolve known issues and create follow-up tickets for new ones

## NVIDIA Skill Combinations
| Purpose | NVIDIA Skill | Reason |
|---------|-------------|--------|
| Correlate video errors | vss-manage-alerts | Cross-reference Sentry errors with VSS video analytics alert events |
| Debug inference failures | nim-agent-blueprint | Investigate NIM inference errors and performance regressions |
| Track deployment issues | aiq-deploy | Monitor AIQ toolkit deployments for runtime errors post-deploy |

## Prerequisites
- `SENTRY_AUTH_TOKEN` with project:read and event:read scopes
- `SENTRY_ORG` — organization slug from Sentry settings

## Security Notes
- Auth token exposes error data which may contain user PII in breadcrumbs
- Stack traces may reveal internal code paths and infrastructure details
- Use organization-scoped tokens; avoid tokens with admin or write permissions for monitoring

## References
- https://github.com/modelcontextprotocol/servers/tree/main/src/sentry
- https://docs.sentry.io/api/
