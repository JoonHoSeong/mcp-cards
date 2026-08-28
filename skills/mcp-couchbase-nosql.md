# Couchbase MCP Server

## Overview
| Field | Value |
|-------|-------|
| **Name** | mcp-couchbase-nosql |
| **Category** | Database |
| **Official** | Official |
| **Source** | https://github.com/Couchbase-Ecosystem/mcp-server-couchbase |
| **Transport** | stdio |
| **Install** | `npx -y mcp-server-couchbase` |

## Description
The Couchbase MCP server provides AI agents with access to Couchbase's distributed NoSQL database, supporting N1QL queries, key-value operations, full-text search, and cluster management. Couchbase's flexible JSON document model combined with built-in caching and search makes it suitable for high-performance applications requiring low-latency reads and flexible querying.

## Key Tools
| Tool | Description |
|------|-------------|
| `query_n1ql` | Execute N1QL (SQL-like) queries against Couchbase |
| `get_document` | Retrieve a document by key from a bucket |
| `upsert_document` | Insert or update a document in a bucket |
| `list_buckets` | List all buckets in the cluster |
| `search_fts` | Perform full-text search queries |
| `get_cluster_info` | Get cluster topology and health information |

## Configuration
```json
{
  "mcpServers": {
    "couchbase": {
      "command": "npx",
      "args": ["-y", "mcp-server-couchbase"],
      "env": {
        "CB_CONNECTION_STRING": "couchbase://localhost",
        "CB_USERNAME": "admin",
        "CB_PASSWORD": "your-password"
      }
    }
  }
}
```

## Use Cases
1. Managing high-throughput document caching for real-time applications
2. Running N1QL analytics queries across distributed JSON datasets
3. Implementing full-text search with relevance scoring for content retrieval

## NVIDIA Skill Combinations
| Purpose | NVIDIA Skill | Reason |
|---------|-------------|--------|
| Document cache for RAG | rag-blueprint | Use Couchbase as a high-speed document cache layer for RAG retrieval pipelines |

## Prerequisites
- Couchbase Server or Couchbase Capella cluster
- `CB_CONNECTION_STRING` with cluster address
- `CB_USERNAME` and `CB_PASSWORD` with appropriate bucket access

## Security Notes
- Use RBAC to restrict bucket and scope access per user/application
- Enable TLS for connection strings in production environments
- Avoid using admin credentials; create application-specific users with minimal permissions

## References
- https://github.com/Couchbase-Ecosystem/mcp-server-couchbase
- https://docs.couchbase.com/
