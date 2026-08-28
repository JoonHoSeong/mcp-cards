# Meilisearch Full-Text Search MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | Meilisearch Full-Text Search MCP Server |
| **Category** | Database/Search |
| **Official** | ⭐ Official / Verified |
| **Source** | https://github.com/meilisearch/mcp-meilisearch |
| **Transport** | stdio |
| **Install** | `npx -y @meilisearch/mcp` |

## Tools & Capabilities
- `search_documents` — Execute typo-tolerant instant search with facet filters and ranking
- `get_index_stats` — Inspect document count, field distribution, and indexing status
- `add_documents` — Ingest document batches with automatic primary key extraction
- `update_settings` — Configure searchable attributes, stop words, and synonym dictionaries
- `get_tasks` — Monitor asynchronous indexing task status and progress

## Client Configuration
```json
{
  "mcpServers": {
    "meilisearch": {
      "command": "npx",
      "args": [
        "-y",
        "@meilisearch/mcp"
      ],
      "env": {
        "MEILISEARCH_HOST": "http://localhost:7700",
        "MEILISEARCH_API_KEY": "YOUR_MASTER_OR_SEARCH_KEY"
      }
    }
  }
}
```

## Security & Best Practices
- Use Search-Only API keys for client queries. Master key is restricted to administrative configuration.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
