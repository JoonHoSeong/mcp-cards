# Qdrant Vector MCP Server

## Overview
| Field | Value |
|-------|-------|
| **Name** | mcp-qdrant-vector |
| **Category** | Database |
| **Official** | Official |
| **Source** | https://github.com/qdrant/mcp-server-qdrant |
| **Transport** | stdio |
| **Install** | `uvx mcp-server-qdrant` |

## Description
The Qdrant MCP server enables AI agents to store, search, and manage vector embeddings in Qdrant collections. It provides semantic memory capabilities through vector similarity search, supporting use cases like RAG pipelines, long-term agent memory, and knowledge retrieval. The server handles collection management, point operations, and filtered vector queries against local or cloud-hosted Qdrant instances.

## Key Tools
| Tool | Description |
|------|-------------|
| `store_memory` | Store text with vector embeddings into a collection |
| `search_memories` | Perform semantic similarity search across stored memories |
| `list_collections` | List all vector collections in the Qdrant instance |
| `create_collection` | Create a new collection with specified vector dimensions |
| `delete_points` | Remove specific points from a collection |
| `get_collection_info` | Get metadata and statistics for a collection |

## Configuration
```json
{
  "mcpServers": {
    "qdrant": {
      "command": "uvx",
      "args": ["mcp-server-qdrant"],
      "env": {
        "QDRANT_URL": "http://localhost:6333",
        "QDRANT_API_KEY": "your-qdrant-api-key",
        "COLLECTION_NAME": "memories"
      }
    }
  }
}
```

## Use Cases
1. Building semantic memory systems for conversational AI agents
2. Implementing vector retrieval for RAG pipelines with filtered search
3. Storing and querying document embeddings for knowledge management

## NVIDIA Skill Combinations
| Purpose | NVIDIA Skill | Reason |
|---------|-------------|--------|
| Vector retrieval backend | nemo-retriever | Serve as the vector store for NeMo Retriever embedding and search pipelines |
| RAG evaluation store | rag-eval | Store evaluation embeddings for measuring retrieval quality and relevance |

## Prerequisites
- Qdrant instance (local Docker or Qdrant Cloud)
- `QDRANT_URL` pointing to the Qdrant REST API endpoint
- `QDRANT_API_KEY` for authenticated access (required for Qdrant Cloud)

## Security Notes
- API keys should be stored securely and rotated periodically
- Use network-level restrictions to limit access to Qdrant endpoints
- Cloud deployments should enable TLS for data in transit

## References
- https://github.com/qdrant/mcp-server-qdrant
- https://qdrant.tech/documentation/
