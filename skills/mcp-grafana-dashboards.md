# Grafana MCP Server

## Overview
| Field | Value |
|-------|-------|
| **Name** | mcp-grafana |
| **Category** | Monitoring |
| **Official** | Yes |
| **Source** | https://github.com/grafana/mcp-grafana |
| **Transport** | stdio |
| **Install** | `npx @grafana/mcp-grafana` |

## Description
Official Grafana MCP server providing AI-assisted access to Grafana dashboards, datasources, alerts, and annotations. Enables programmatic querying of monitoring data, dashboard discovery, and alert management through natural language interactions with Grafana instances.

## Key Tools
| Tool | Description |
|------|-------------|
| `list_dashboards` | List all available Grafana dashboards |
| `get_dashboard` | Retrieve dashboard details and panels by UID |
| `query_datasource` | Execute queries against configured datasources |
| `list_alerts` | List configured alert rules |
| `get_alert_status` | Get current firing status of alerts |
| `search_annotations` | Search dashboard annotations by time range |
| `list_datasources` | List all configured datasources |
| `create_annotation` | Create annotations on dashboards |

## Configuration
```json
{
  "mcpServers": {
    "grafana": {
      "command": "npx",
      "args": ["@grafana/mcp-grafana"],
      "env": {
        "GRAFANA_URL": "http://localhost:3000",
        "GRAFANA_TOKEN": "your-service-account-token"
      }
    }
  }
}
```

## Use Cases
1. Query and visualize video analytics pipeline metrics from Grafana dashboards
2. Monitor alert status for GPU inference workloads and auto-annotate incidents
3. Discover datasources and run ad-hoc PromQL/Loki queries through AI assistance

## NVIDIA Skill Combinations
| Purpose | NVIDIA Skill | Reason |
|---------|-------------|--------|
| Video analytics dashboards | vss-manage-alerts | Monitor VSS video analytics metrics and alert states through Grafana |

## Prerequisites
- Grafana 9.0+ instance running and accessible
- `GRAFANA_URL` — Grafana instance URL
- `GRAFANA_TOKEN` — Service account token with appropriate permissions

## Security Notes
- Use service account tokens with minimal required permissions (Viewer role for read-only use)
- Never expose Grafana admin tokens; prefer scoped API keys per use case

## References
- https://github.com/grafana/mcp-grafana
- https://grafana.com/docs/grafana/latest/developers/http_api/
