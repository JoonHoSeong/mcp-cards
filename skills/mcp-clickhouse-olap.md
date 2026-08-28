# ClickHouse MCP Server

## Overview
| Field | Value |
|-------|-------|
| **Name** | mcp-clickhouse-olap |
| **Category** | Database |
| **Official** | Official (Altinity) |
| **Source** | https://github.com/Altinity/altinity-mcp |
| **Transport** | stdio |
| **Install** | `uvx altinity-mcp` |

## Description
The ClickHouse MCP server (by Altinity) provides AI agents with access to ClickHouse's columnar OLAP database for high-performance analytical queries. It supports SQL execution, schema inspection, query explanation, and table statistics, making it ideal for real-time analytics, time-series data, log analysis, and large-scale aggregation workloads where sub-second query latency on billions of rows is required.

## Key Tools
| Tool | Description |
|------|-------------|
| `execute_query` | Execute analytical SQL queries against ClickHouse |
| `list_tables` | List all tables in a database |
| `describe_table` | Get column definitions and engine details for a table |
| `get_table_stats` | Retrieve row counts, storage size, and partition info |
| `explain_query` | Get the query execution plan for optimization |
| `list_databases` | List all databases on the ClickHouse server |

## Configuration
```json
{
  "mcpServers": {
    "clickhouse": {
      "command": "uvx",
      "args": ["altinity-mcp"],
      "env": {
        "CLICKHOUSE_URL": "http://localhost:8123",
        "CLICKHOUSE_USER": "default",
        "CLICKHOUSE_PASSWORD": "your-password"
      }
    }
  }
}
```

## Use Cases
1. Real-time analytics dashboards over billions of event records
2. Query optimization and performance analysis for OLAP workloads
3. Time-series analysis for IoT sensor data and video analytics metrics

## NVIDIA Skill Combinations
| Purpose | NVIDIA Skill | Reason |
|---------|-------------|--------|
| OLAP for video analytics | vss-query-analytics | Store and query aggregated video analytics metrics with sub-second latency |

## Prerequisites
- ClickHouse server (self-hosted or ClickHouse Cloud)
- `CLICKHOUSE_URL` HTTP interface endpoint
- `CLICKHOUSE_USER` and `CLICKHOUSE_PASSWORD` for authentication

## Security Notes
- Use read-only users for analytics queries to prevent accidental data modification
- Enable SSL/TLS for remote connections to ClickHouse Cloud
- Restrict query execution time and memory limits per user profile

## References
- https://github.com/Altinity/altinity-mcp
- https://clickhouse.com/docs
