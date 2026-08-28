# Grafana Loki MCP Server

## Overview
| Field | Value |
|-------|-------|
| **Name** | loki-mcp |
| **Category** | Logging |
| **Official** | Yes |
| **Source** | https://github.com/grafana/loki-mcp |
| **Transport** | stdio |
| **Install** | `npx @grafana/loki-mcp` |

## Description
Official Grafana Loki MCP server for querying and exploring log streams using LogQL. Provides AI-assisted log analysis, label discovery, and real-time log tailing capabilities for debugging distributed systems and monitoring application behavior.

## Key Tools
| Tool | Description |
|------|-------------|
| `query_logs` | Execute LogQL queries against Loki |
| `get_labels` | List all available log labels |
| `get_label_values` | Get values for a specific label |
| `query_range` | Query logs over a time range with step intervals |
| `get_series` | Get log series matching label matchers |
| `tail_logs` | Stream live log entries in real-time |

## Configuration
```json
{
  "mcpServers": {
    "loki": {
      "command": "npx",
      "args": ["@grafana/loki-mcp"],
      "env": {
        "LOKI_URL": "http://localhost:3100"
      }
    }
  }
}
```

## Use Cases
1. Query and analyze GPU training job logs from Kubernetes clusters
2. Tail real-time logs from inference services to debug model loading issues
3. Explore log labels and series to build effective LogQL queries for pipeline monitoring

## NVIDIA Skill Combinations
| Purpose | NVIDIA Skill | Reason |
|---------|-------------|--------|
| Training logs | tao-run-on-kubernetes | Query and analyze TAO training job logs from Kubernetes pods |

## Prerequisites
- Grafana Loki instance running and accessible
- `LOKI_URL` — Loki HTTP API endpoint

## Security Notes
- Restrict Loki access to authorized networks; avoid exposing the API publicly
- Use authentication proxies or Grafana Cloud credentials for production deployments

## References
- https://github.com/grafana/loki-mcp
- https://grafana.com/docs/loki/latest/reference/loki-http-api/
