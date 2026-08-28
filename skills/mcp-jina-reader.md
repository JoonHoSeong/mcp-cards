# Jina AI Reader & Reranker MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | Jina AI Reader & Reranker MCP Server |
| **Category** | Search/Extraction |
| **Official** | ⭐ Official (Jina AI) |
| **Source** | https://github.com/jina-ai/mcp-server |
| **Transport** | stdio |
| **Install** | `npx -y @jina-ai/mcp-server` |

## Tools & Capabilities
- `read_url` — Convert any webpage URL into clean, token-efficient Markdown via `r.jina.ai`
- `search_web` — Execute web search with direct Markdown passage extraction via `s.jina.ai`
- `rerank_documents` — High-accuracy cross-encoder neural reranker via `rerank.jina.ai`
- `embed_texts` — Generate 8192-token long-context embeddings via Jina Embeddings v3
- `ground_fact` — Check factual consistency of claims against live search results

## Client Configuration
```json
{
  "mcpServers": {
    "jina": {
      "command": "npx",
      "args": [
        "-y",
        "@jina-ai/mcp-server"
      ],
      "env": {
        "JINA_API_KEY": "YOUR_JINA_API_KEY"
      }
    }
  }
}
```

## Security & Best Practices
- Clean stripping of tracking scripts and ads. Safe fallback for paywalled/cookie-walled pages.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
