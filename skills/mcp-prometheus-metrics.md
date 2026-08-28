# Prometheus MCP Server

## Overview
| Field | Value |
|-------|-------|
| **Name** | prometheus-mcp-server |
| **Category** | Metrics |
| **Official** | Community |
| **Source** | https://github.com/pab1it0/prometheus-mcp-server |
| **Transport** | stdio |
| **Install** | `pip install prometheus-mcp-server` |

## Description
Community Prometheus MCP server for executing PromQL queries, discovering metrics, and monitoring target health. Enables AI-assisted metrics exploration, instant and range queries, alert rule inspection, and label discovery across Prometheus-monitored infrastructure.

## Key Tools
| Tool | Description |
|------|-------------|
| `query_instant` | Execute an instant PromQL query |
| `query_range` | Execute a range PromQL query with step intervals |
| `list_metrics` | List all available metric names |
| `get_targets` | Get scrape target health status |
| `get_alerts` | List currently firing alerts |
| `get_rules` | Get configured alerting and recording rules |
| `list_labels` | List all label names in the TSDB |

## Configuration
```json
{
  "mcpServers": {
    "prometheus": {
      "command": "python",
      "args": ["-m", "prometheus_mcp_server"],
      "env": {
        "PROMETHEUS_URL": "http://localhost:9090"
      }
    }
  }
}
```

## Use Cases
1. Query GPU utilization and memory metrics from Jetson edge devices
2. Monitor inference latency percentiles and throughput across model endpoints
3. Discover available metrics and build PromQL expressions for capacity planning

## NVIDIA Skill Combinations
| Purpose | NVIDIA Skill | Reason |
|---------|-------------|--------|
| Edge device metrics | jetson-diagnostic | Monitor Jetson device health metrics via Prometheus exporters |

## Prerequisites
- Prometheus server running and accessible
- `PROMETHEUS_URL` — Prometheus HTTP API endpoint
- Python 3.10+ for the MCP server

## Security Notes
- Prometheus API is unauthenticated by default; use reverse proxy with auth for production
- Restrict network access to Prometheus to authorized clients only

## References
- https://github.com/pab1it0/prometheus-mcp-server
- https://prometheus.io/docs/prometheus/latest/querying/api/
