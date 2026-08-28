# Chroma Vector Database MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | Chroma Vector Database MCP Server |
| **Category** | Database/Vector |
| **Official** | 커뮤니티 검증 |
| **Source** | https://github.com/chroma-core/chroma-mcp |
| **Transport** | stdio |
| **Install** | `npx -y chroma-mcp` |

## Tools & Capabilities
- `list_collections` — List Chroma collections, distance functions, and counts
- `query_collection` — Query nearest neighbors using text queries or raw embeddings
- `add_documents` — Ingest text documents with automatic local embedding generation
- `update_metadata` — Modify metadata dictionaries attached to specific document IDs
- `delete_documents` — Remove document vectors by ID list or metadata filter

## Client Configuration
```json
{
  "mcpServers": {
    "chroma": {
      "command": "npx",
      "args": [
        "-y",
        "chroma-mcp"
      ],
      "env": {
        "CHROMA_SERVER_HOST": "localhost",
        "CHROMA_SERVER_HTTP_PORT": "8000"
      }
    }
  }
}
```

## Security & Best Practices
- Local sandbox storage isolation. Ensure embedding dimension matches collection config.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
