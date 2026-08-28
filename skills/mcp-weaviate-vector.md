# Weaviate Vector MCP Server

## Overview
| Field | Value |
|-------|-------|
| **Name** | mcp-weaviate-vector |
| **Category** | Database |
| **Official** | Official (built-in) |
| **Source** | https://github.com/weaviate/mcp-server-weaviate |
| **Transport** | stdio |
| **Install** | `npx -y mcp-server-weaviate` |

## Description
The Weaviate MCP server provides AI agents with access to Weaviate's vector database capabilities, including hybrid search that combines vector similarity with keyword matching. It supports schema management, object CRUD operations, and advanced query modes, making it well-suited for RAG applications that benefit from both semantic and lexical retrieval strategies.

## Key Tools
| Tool | Description |
|------|-------------|
| `query_objects` | Query objects with filters and property selection |
| `add_objects` | Add new objects with optional vector embeddings |
| `search_vectors` | Perform pure vector similarity search |
| `get_schema` | Retrieve the current database schema and classes |
| `create_class` | Create a new class with properties and vectorizer config |
| `hybrid_search` | Combine vector and keyword search with configurable alpha |

## Configuration
```json
{
  "mcpServers": {
    "weaviate": {
      "command": "npx",
      "args": ["-y", "mcp-server-weaviate"],
      "env": {
        "WEAVIATE_URL": "http://localhost:8080",
        "WEAVIATE_API_KEY": "your-weaviate-api-key"
      }
    }
  }
}
```

## Use Cases
1. Implementing hybrid search combining semantic and keyword retrieval for RAG
2. Managing vectorized knowledge bases with schema-driven data models
3. Building multi-tenant search applications with class-level isolation

## NVIDIA Skill Combinations
| Purpose | NVIDIA Skill | Reason |
|---------|-------------|--------|
| Hybrid search for RAG | rag-blueprint | Leverage Weaviate's hybrid search as the retrieval backend for NVIDIA RAG pipelines |

## Prerequisites
- Weaviate instance (local Docker, Weaviate Cloud Services, or embedded)
- `WEAVIATE_URL` endpoint
- `WEAVIATE_API_KEY` for authenticated deployments

## Security Notes
- Enable authentication and RBAC for multi-user deployments
- Use TLS for production connections to prevent data interception
- Restrict vectorizer module access to prevent unauthorized embedding generation

## References
- https://github.com/weaviate/mcp-server-weaviate
- https://weaviate.io/developers/weaviate
