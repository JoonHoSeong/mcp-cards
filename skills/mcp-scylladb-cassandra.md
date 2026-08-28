# ScyllaDB / Cassandra MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | ScyllaDB / Cassandra MCP Server |
| **Category** | Database/NoSQL |
| **Official** | ⭐ Official (ScyllaDB) |
| **Source** | https://github.com/scylladb/mcp-scylladb |
| **Transport** | stdio |
| **Install** | `npx -y @scylladb/mcp-server` |

## Tools & Capabilities
- `describe_keyspaces` — List keyspaces, replication strategies, and durability settings
- `describe_tables` — Inspect partition keys, clustering columns, and data types
- `execute_cql_query` — Execute CQL queries with consistency level configuration
- `explain_cql_plan` — Analyze trace and execution profile for partition queries
- `inspect_node_status` — Check cluster topology, token rings, and latency stats

## Client Configuration
```json
{
  "mcpServers": {
    "scylladb": {
      "command": "npx",
      "args": [
        "-y",
        "@scylladb/mcp-server"
      ],
      "env": {
        "SCYLLA_CONTACT_POINTS": "127.0.0.1",
        "SCYLLA_KEYSPACE": "my_keyspace",
        "SCYLLA_USER": "cassandra",
        "SCYLLA_PASSWORD": "cassandra"
      }
    }
  }
}
```

## Security & Best Practices
- Enforce explicit consistency levels (e.g. `LOCAL_QUORUM`). Warn on unfiltered `ALLOW FILTERING` scans.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
