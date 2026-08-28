# Mixpanel Product Analytics MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | Mixpanel Product Analytics MCP Server |
| **Category** | Analytics/Product |
| **Official** | ⭐ Official / Verified |
| **Source** | https://github.com/mixpanel/mcp-mixpanel |
| **Transport** | stdio |
| **Install** | `npx -y @mixpanel/mcp-server` |

## Tools & Capabilities
- `query_jql` — Execute JavaScript-based JQL queries for advanced user behavior data mining
- `get_retention_tables` — Retrieve N-day user cohort retention curves and churn rates
- `export_events` — Stream raw historical events within specific date range
- `query_user_profiles` — Inspect individual user properties, lifetime value, and engagement
- `list_bookmarks` — Query saved reports, funnels, and retention dashboards

## Client Configuration
```json
{
  "mcpServers": {
    "mixpanel": {
      "command": "npx",
      "args": [
        "-y",
        "@mixpanel/mcp-server"
      ],
      "env": {
        "MIXPANEL_SERVICE_ACCOUNT_USER": "service_account@...",
        "MIXPANEL_SERVICE_ACCOUNT_SECRET": "YOUR_SECRET",
        "MIXPANEL_PROJECT_ID": "123456"
      }
    }
  }
}
```

## Security & Best Practices
- Service account authentication with read-only analytics permissions.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
