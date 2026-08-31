# Semantic Scholar Academic Research MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | Semantic Scholar Academic Research MCP Server |
| **Category** | Academic/Research |
| **Official** | ⭐ Official (Allen Institute for AI) |
| **Source** | https://github.com/allenai/mcp-semantic-scholar |
| **Transport** | stdio |
| **Install** | `npx -y @allenai/mcp-semantic-scholar` |

## Tools & Capabilities
- `search_academic_papers` — Query 200M+ research papers with citation count and influential citations
- `get_citation_graph` — Traverse paper reference edges, citations, and literature genealogy
- `get_author_profile` — Inspect author h-index, top publications, and co-author networks
- `get_paper_tldr` — Retrieve concise AI-generated TLDR summaries of complex academic papers

## Client Configuration
```json
{
  "mcpServers": {
    "semantic-scholar": {
      "command": "npx",
      "args": [
        "-y",
        "@allenai/mcp-semantic-scholar"
      ],
      "env": {
        "SEMANTIC_SCHOLAR_API_KEY": "YOUR_API_KEY"
      }
    }
  }
}
```

## Security & Best Practices
- API Key authentication. Caches citation metadata in memory.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
