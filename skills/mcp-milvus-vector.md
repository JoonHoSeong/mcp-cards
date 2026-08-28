# Milvus Vector Database MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | Milvus Vector Database MCP Server |
| **Category** | Database/Vector |
| **Official** | ⭐ Official (Zilliz / Milvus) |
| **Source** | https://github.com/zilliztech/mcp-server-milvus |
| **Transport** | stdio |
| **Install** | `npx -y @zilliz/mcp-server-milvus` |

## Tools & Capabilities
- `list_collections` — List Milvus collections, dimensions, and index types (HNSW, IVF_FLAT)
- `vector_search` — Perform high-speed ANN vector similarity search with metadata filtering
- `hybrid_search` — Execute multi-vector and dense+sparse hybrid retrieval
- `insert_vectors` — Ingest vector embeddings alongside structured payload metadata
- `manage_partitions` — Create, load, and release collection partitions in memory

## Client Configuration
```json
{
  "mcpServers": {
    "milvus": {
      "command": "npx",
      "args": [
        "-y",
        "@zilliz/mcp-server-milvus"
      ],
      "env": {
        "MILVUS_URI": "http://localhost:19530",
        "MILVUS_TOKEN": "root:Milvus"
      }
    }
  }
}
```

## Security & Best Practices
- Enforce collection partition loading rules. Validate embedding dimensionality before insert.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
