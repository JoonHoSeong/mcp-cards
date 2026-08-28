# Upstash MCP Server

## Overview
| Field | Value |
|-------|-------|
| **Name** | Upstash MCP |
| **Category** | Serverless DB |
| **Official** | Yes (Upstash) |
| **Source** | https://github.com/upstash/mcp-server |
| **Transport** | stdio |
| **Install** | `npx @upstash/mcp-server` |

## Description
The Upstash MCP Server provides AI agents with access to Upstash's serverless Redis, QStash message queue, and Workflow orchestration services. It enables low-latency caching, vector search, scheduled task execution, and durable workflow management — all with pay-per-request pricing and no infrastructure to manage.

## Key Tools
| Tool | Description |
|------|-------------|
| `redis_get` | Get a value from Upstash Redis |
| `redis_set` | Set a key-value pair in Redis |
| `redis_search` | Perform vector similarity search |
| `qstash_publish` | Publish a message to QStash |
| `qstash_schedule` | Schedule a recurring task |
| `workflow_run` | Trigger a durable workflow execution |
| `workflow_status` | Check workflow execution status |
| `list_databases` | List all Redis databases |

## Configuration
```json
{
  "mcpServers": {
    "upstash": {
      "command": "npx",
      "args": ["@upstash/mcp-server"],
      "env": {
        "UPSTASH_REDIS_URL": "https://your-redis.upstash.io",
        "UPSTASH_TOKEN": "your-rest-token"
      }
    }
  }
}
```

## Use Cases
1. Cache RAG embeddings and query results for sub-millisecond retrieval
2. Schedule periodic model evaluation jobs via QStash
3. Orchestrate multi-step AI pipelines with durable workflow guarantees

## NVIDIA Skill Combinations
| Purpose | NVIDIA Skill | Reason |
|---------|-------------|--------|
| Vector caching | rag-blueprint | Cache embedding vectors in serverless Redis for fast retrieval |
| Query result cache | rag-blueprint | Store frequent RAG responses to reduce inference calls |
| Pipeline scheduling | rag-blueprint | Schedule periodic index refresh via QStash |

## Prerequisites
- Upstash account with Redis database created
- `UPSTASH_REDIS_URL` REST endpoint URL
- `UPSTASH_TOKEN` for authentication

## Security Notes
- Use read-only tokens for query-only operations
- Enable TLS encryption (default for Upstash REST API)
- Set TTL on cached data to prevent stale responses
- Monitor usage through Upstash console for anomalies

## References
- https://github.com/upstash/mcp-server
- https://docs.upstash.com/redis
- https://docs.upstash.com/qstash
