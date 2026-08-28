# CockroachDB MCP Server

## Overview
| Field | Value |
|-------|-------|
| **Name** | mcp-cockroachdb-distributed |
| **Category** | Database |
| **Official** | Community |
| **Source** | https://github.com/amineelkouhen/mcp-cockroachdb |
| **Transport** | stdio |
| **Install** | `npx -y mcp-cockroachdb` |

## Description
The CockroachDB MCP server provides AI agents with access to CockroachDB's distributed SQL database, offering PostgreSQL-compatible queries with automatic horizontal scaling, multi-region replication, and strong consistency guarantees. It supports SQL execution, schema inspection, cluster health monitoring, and range distribution analysis, making it suitable for applications requiring global distribution and high availability.

## Key Tools
| Tool | Description |
|------|-------------|
| `execute_sql` | Execute SQL statements against the CockroachDB cluster |
| `list_databases` | List all databases in the cluster |
| `list_tables` | List tables within a specific database |
| `get_schema` | Get detailed schema for tables including constraints |
| `get_cluster_status` | Check cluster health, node status, and replication |
| `get_ranges_info` | Inspect range distribution across nodes |

## Configuration
```json
{
  "mcpServers": {
    "cockroachdb": {
      "command": "npx",
      "args": ["-y", "mcp-cockroachdb"],
      "env": {
        "COCKROACH_URL": "postgresql://user:pass@localhost:26257/defaultdb?sslmode=verify-full"
      }
    }
  }
}
```

## Use Cases
1. Managing globally distributed databases with multi-region consistency
2. Monitoring cluster health and range distribution for operational awareness
3. Running transactional workloads that require serializable isolation at scale

## NVIDIA Skill Combinations
| Purpose | NVIDIA Skill | Reason |
|---------|-------------|--------|
| Distributed state management | physical-ai-infrastructure | Store distributed state for physical AI systems requiring multi-region consistency and fault tolerance |

## Prerequisites
- CockroachDB cluster (self-hosted or CockroachDB Cloud)
- `COCKROACH_URL` PostgreSQL-compatible connection string
- TLS certificates for secure connections

## Security Notes
- Always use `sslmode=verify-full` for production connections
- Create application-specific SQL users with GRANT-based access control
- Monitor range distribution to prevent hotspots and data skew

## References
- https://github.com/amineelkouhen/mcp-cockroachdb
- https://www.cockroachlabs.com/docs/
