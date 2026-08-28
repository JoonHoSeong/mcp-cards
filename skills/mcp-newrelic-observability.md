# New Relic MCP Server

## Overview
| Field | Value |
|-------|-------|
| **Name** | newrelic-mcp-server |
| **Category** | Observability |
| **Official** | Community |
| **Source** | https://github.com/piekstra/newrelic-mcp-server |
| **Transport** | stdio |
| **Install** | `pip install newrelic-mcp-server` |

## Description
Community New Relic MCP server for querying telemetry data via NRQL, managing entities, and monitoring application performance. Enables AI-assisted observability workflows including dashboard exploration, alert inspection, and synthetic monitor management across the New Relic platform.

## Key Tools
| Tool | Description |
|------|-------------|
| `query_nrql` | Execute NRQL queries against New Relic data |
| `list_accounts` | List accessible New Relic accounts |
| `get_entity` | Get details for a monitored entity |
| `list_dashboards` | List available dashboards |
| `get_alerts` | Get alert conditions and violations |
| `list_synthetic_monitors` | List configured synthetic monitors |

## Configuration
```json
{
  "mcpServers": {
    "newrelic": {
      "command": "python",
      "args": ["-m", "newrelic_mcp_server"],
      "env": {
        "NEW_RELIC_API_KEY": "your-user-api-key",
        "NEW_RELIC_ACCOUNT_ID": "your-account-id"
      }
    }
  }
}
```

## Use Cases
1. Query inference service response times and error rates via NRQL
2. Monitor model endpoint health using synthetic monitors and alert policies
3. Explore application entities and correlate performance regressions with deployments

## NVIDIA Skill Combinations
| Purpose | NVIDIA Skill | Reason |
|---------|-------------|--------|
| Monitor inference | tao-run-inference-service | Observe TAO inference service performance and health in New Relic |

## Prerequisites
- New Relic account with data reporting
- `NEW_RELIC_API_KEY` — User API key with NRQL query permissions
- `NEW_RELIC_ACCOUNT_ID` — Target account identifier

## Security Notes
- Use User API keys scoped to specific accounts rather than License keys
- Restrict API key permissions to query-only for monitoring use cases

## References
- https://github.com/piekstra/newrelic-mcp-server
- https://docs.newrelic.com/docs/apis/nerdgraph/get-started/introduction-new-relic-nerdgraph/
