# Supabase pgvector MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | Supabase pgvector MCP Server |
| **Category** | Database/Vector |
| **Official** | ⭐ Official (Supabase) |
| **Source** | https://github.com/supabase/mcp-pgvector |
| **Transport** | stdio |
| **Install** | `npx -y @supabase/mcp-pgvector` |

## Tools & Capabilities
- `match_embeddings` — Execute cosine/L2/inner product distance similarity queries
- `hybrid_search` — Combine dense vector similarity with Postgres tsvector full-text search
- `inspect_hnsw_indexes` — Audit HNSW and IVFFlat index build parameters and memory
- `create_vector_table` — Scaffold vector storage table with automatic embedding trigger
- `vacuum_vector_index` — Run maintenance and index optimization routines

## Client Configuration
```json
{
  "mcpServers": {
    "supabase-pgvector": {
      "command": "npx",
      "args": [
        "-y",
        "@supabase/mcp-pgvector"
      ],
      "env": {
        "SUPABASE_DB_URL": "postgresql://postgres:password@db.supabase.co:5432/postgres"
      }
    }
  }
}
```

## Security & Best Practices
- Enforce PostgreSQL Row Level Security (RLS). Parameterize vector queries to prevent SQL injection.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
