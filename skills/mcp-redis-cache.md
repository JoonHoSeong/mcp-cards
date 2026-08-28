# Redis MCP Server

## Overview
| Field | Value |
|-------|-------|
| **Name** | Redis MCP Server |
| **Category** | Database/Cache |
| **Official** | ⭐ Official (Redis Labs) |
| **Source** | https://github.com/redis/mcp-redis |
| **Transport** | stdio |
| **Install** | `npx -y @redis/mcp` |

## Description
Official Redis MCP server providing natural language access to Redis data structures, vector search, JSON documents, streams, and hash operations. Enables AI agents to manage caches, perform vector similarity searches, handle session data, and process real-time streams through the MCP protocol.

## Key Tools
| Tool | Description |
|------|-------------|
| `set` | Set a string key-value pair with optional TTL |
| `get` | Get the value of a string key |
| `delete` | Delete one or more keys |
| `search` | Perform vector similarity or full-text search |
| `list_indexes` | List all RediSearch indexes |
| `create_index` | Create a new search index with schema definition |
| `aggregate` | Run aggregation queries on indexed data |
| `json_set` | Set a JSON value at a path in a key |
| `json_get` | Get a JSON value at a path from a key |
| `stream_add` | Add entries to a Redis Stream |
| `stream_read` | Read entries from a Redis Stream |
| `hash_set` | Set fields in a hash |
| `hash_get` | Get fields from a hash |

## Configuration
```json
{
  "mcpServers": {
    "redis": {
      "command": "npx",
      "args": ["-y", "@redis/mcp"],
      "env": {
        "REDIS_URL": "redis://localhost:6379"
      }
    }
  }
}
```

## Use Cases
1. Cache management via natural language — set, get, invalidate cache entries
2. Vector search queries for semantic similarity over embeddings
3. Session management — create, read, expire user sessions
4. Real-time feature store for ML inference pipelines
5. Stream processing — publish and consume event streams

## NVIDIA Skill Combinations
| Purpose | NVIDIA Skill | Reason |
|---------|-------------|--------|
| Embedding cache layer | `rag-blueprint` | Cache computed embeddings to reduce re-computation in RAG pipelines |
| Real-time event cache | `vss-query-analytics` | Buffer and cache analytics events for real-time vector search queries |
| Retrieval cache layer | `nemo-retriever` | Cache frequently retrieved passages to accelerate inference |

## Prerequisites
- Redis server 7.0+ (for vector search) or Redis Cloud account
- Connection URL (e.g., `redis://localhost:6379` or Redis Cloud endpoint)
- RediSearch module for vector/full-text search capabilities
- RedisJSON module for JSON document operations

## Security Notes
- Use ACL-restricted user with minimal permissions for the agent
- Enable TLS for cloud/remote connections
- Avoid exposing Redis port publicly — use SSH tunnel or VPN
- Set `maxmemory-policy` to prevent OOM in production
- Use separate Redis databases or key prefixes for isolation

## References
- [GitHub Repository](https://github.com/redis/mcp-redis)
- [Redis Vector Search Documentation](https://redis.io/docs/interact/search-and-query/search/vectors/)
- [Redis Cloud](https://redis.com/cloud/)
