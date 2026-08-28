# MongoDB Atlas MCP Server

## Overview
| Field | Value |
|-------|-------|
| **Name** | mcp-mongodb-atlas |
| **Category** | Database |
| **Official** | Official |
| **Source** | https://github.com/mongodb-js/mongodb-mcp-server |
| **Transport** | stdio |
| **Install** | `npx -y mongodb-mcp-server` |

## Description
The MongoDB Atlas MCP server provides full-featured access to MongoDB Atlas clusters, enabling document CRUD operations, aggregation pipelines, collection management, index creation, and Atlas Search capabilities. It supports both local MongoDB instances and cloud-hosted Atlas deployments, making it suitable for document-oriented application development and vector search workloads.

## Key Tools
| Tool | Description |
|------|-------------|
| `find_documents` | Query documents with filters, projections, and sort options |
| `insert_document` | Insert one or more documents into a collection |
| `update_document` | Update documents matching a filter with specified modifications |
| `aggregate` | Run aggregation pipelines for complex data transformations |
| `list_collections` | List all collections in the connected database |
| `create_index` | Create indexes for query optimization |
| `atlas_search` | Perform full-text and vector search using Atlas Search |

## Configuration
```json
{
  "mcpServers": {
    "mongodb-atlas": {
      "command": "npx",
      "args": ["-y", "mongodb-mcp-server"],
      "env": {
        "MONGODB_URI": "mongodb+srv://user:pass@cluster.mongodb.net/dbname",
        "ATLAS_API_PUBLIC_KEY": "your-public-key",
        "ATLAS_API_PRIVATE_KEY": "your-private-key"
      }
    }
  }
}
```

## Use Cases
1. Building and managing document-oriented application backends
2. Running complex aggregation pipelines for data analytics
3. Implementing vector search for RAG applications with Atlas Search

## NVIDIA Skill Combinations
| Purpose | NVIDIA Skill | Reason |
|---------|-------------|--------|
| Document store for RAG | rag-blueprint | Store and retrieve document embeddings for retrieval-augmented generation pipelines |

## Prerequisites
- MongoDB Atlas account or local MongoDB instance
- `MONGODB_URI` connection string
- Atlas API credentials (public/private key pair) for cluster management operations

## Security Notes
- Store connection strings and API keys in environment variables, never in config files
- Use database-level access controls and IP allowlists in Atlas
- Restrict API key permissions to minimum required scope

## References
- https://github.com/mongodb-js/mongodb-mcp-server
- https://www.mongodb.com/docs/atlas/
