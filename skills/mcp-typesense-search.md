# Typesense Typo-Tolerant Search MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | Typesense Typo-Tolerant Search MCP Server |
| **Category** | Database/Search |
| **Official** | 커뮤니티 검증 |
| **Source** | https://github.com/typesense/mcp-typesense |
| **Transport** | stdio |
| **Install** | `npx -y typesense-mcp` |

## Tools & Capabilities
- `multi_search` — Execute concurrent multi-collection search queries with facet filters
- `describe_collection` — Inspect collection schema, tokenizers, and vector embeddings
- `index_documents` — Upsert document batches into Typesense collection
- `vector_search` — Perform hybrid keyword and embedding nearest neighbor search
- `manage_curations` — Create search overrides and pin promoted result items

## Client Configuration
```json
{
  "mcpServers": {
    "typesense": {
      "command": "npx",
      "args": [
        "-y",
        "typesense-mcp"
      ],
      "env": {
        "TYPESENSE_NODES": "http://localhost:8108",
        "TYPESENSE_API_KEY": "YOUR_TYPESENSE_API_KEY"
      }
    }
  }
}
```

## Security & Best Practices
- Isolate scoped search keys with embedded filters for multi-tenant isolation.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
