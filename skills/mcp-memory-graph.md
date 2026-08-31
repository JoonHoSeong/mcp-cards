# Anthropic Memory Knowledge Graph MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | Anthropic Memory MCP Server |
| **Category** | AI/Memory |
| **Official** | ⭐ Official (Anthropic Reference) |
| **Source** | https://github.com/modelcontextprotocol/servers/tree/main/src/memory |
| **Transport** | stdio |
| **Install** | `npx -y @modelcontextprotocol/server-memory` |

## Tools & Capabilities
- `create_entities` — Create multiple new entities in the persistent knowledge graph
- `create_relations` — Create directed relationship edges between existing entities
- `add_observations` — Append factual observations or timestamped memory logs to existing entities
- `delete_entities` — Remove entities and cascade-delete connected relations from graph
- `delete_observations` — Remove specific outdated or invalidated factual observations
- `read_graph` — Retrieve the entire serialized knowledge graph structure
- `search_nodes` — Search entities and relations by name, category, or semantic keyword

## Client Configuration
```json
{
  "mcpServers": {
    "memory": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-memory"]
    }
  }
}
```

## Security & Best Practices
- Local JSON memory file persistence. Graph memory is isolated per user profile and never transmitted externally without explicit user instructions.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| Persistent Agent Memory | `nvidia-skill-finder` / `nemotron-customize` | Retain multi-turn architectural decisions and user preferences across distinct coding sessions |
| Complex Knowledge Graph RAG | `rag-blueprint` / `cosmos-dataset-search` | Graph-augmented hybrid retrieval combining dense vector embeddings with persistent entity relations |
