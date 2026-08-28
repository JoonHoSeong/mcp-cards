# Datadog MCP Server

## Overview
| Field | Value |
|-------|-------|
| **Name** | Datadog MCP Server |
| **Category** | Monitoring/Observability |
| **Official** | ⭐ Official (Datadog Labs) |
| **Source** | https://github.com/datadog-labs/mcp-server |
| **Transport** | stdio |
| **Install** | `npx -y @datadog/mcp` |

## Description
Official Datadog MCP server that provides AI agents with access to Datadog's full observability platform. Query metrics, search logs, manage monitors, investigate incidents, and analyze APM traces directly from your development environment without switching to the Datadog UI.

## Key Tools
| Tool | Description |
|------|-------------|
| `query_metrics` | Query time-series metrics with Datadog query syntax |
| `search_logs` | Search and filter log entries across services |
| `list_monitors` | List all configured monitors and their states |
| `get_monitor_status` | Get detailed status of a specific monitor |
| `list_incidents` | List active and resolved incidents |
| `get_trace` | Retrieve a specific distributed trace |
| `search_apm` | Search APM traces by service, operation, or tags |
| `query_rum` | Query Real User Monitoring data |
| `list_synthetics` | List synthetic test configurations and results |
| `create_monitor` | Create a new monitor with alert conditions |
| `mute_monitor` | Mute a monitor for a specified duration |

## Configuration
```json
{
  "mcpServers": {
    "datadog": {
      "command": "npx",
      "args": ["-y", "@datadog/mcp"],
      "env": {
        "DD_API_KEY": "your-api-key-here",
        "DD_APP_KEY": "your-app-key-here",
        "DD_SITE": "datadoghq.com"
      }
    }
  }
}
```

## Use Cases
1. Query logs and metrics from the IDE during debugging sessions
2. Investigate incidents and correlate across services
3. Create and configure monitors for new deployments
4. Analyze APM traces to identify performance bottlenecks
5. Check synthetic test results after infrastructure changes
6. Debug performance issues with real-time metric queries

## NVIDIA Skill Combinations
| Purpose | NVIDIA Skill | Reason |
|---------|-------------|--------|
| Video analytics monitoring | vss-manage-alerts | Monitor VSS pipeline health metrics and alert on degradation |
| Training job monitoring | tao-run-on-kubernetes | Track GPU utilization, memory, and training progress metrics |
| Edge device monitoring | jetson-diagnostic | Correlate Jetson device metrics with centralized Datadog dashboards |

## Prerequisites
- Datadog account (any tier)
- `DD_API_KEY` — API key from Organization Settings
- `DD_APP_KEY` — Application key with appropriate scopes
- Node.js 18+ for npx execution

## Security Notes
- Use scoped Application Keys with minimal permissions (read-only by default)
- API Keys grant data submission access — protect accordingly
- Application Keys inherit the permissions of the creating user; use a service account
- Separate keys for development vs. production environments
- Monitor creation/mutation requires explicit write scopes — disable if not needed

## References
- https://github.com/datadog-labs/mcp-server
- https://docs.datadoghq.com/api/latest/
- https://docs.datadoghq.com/account_management/api-app-keys/
- https://docs.datadoghq.com/monitors/
