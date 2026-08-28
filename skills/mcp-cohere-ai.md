# Cohere Command & Embed MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | Cohere Command & Embed MCP Server |
| **Category** | AI/LLM |
| **Official** | ⭐ Official (Cohere) |
| **Source** | https://github.com/cohere-ai/cohere-mcp |
| **Transport** | stdio |
| **Install** | `npx -y @cohere/mcp-server` |

## Tools & Capabilities
- `chat` — Generate responses using Command R / Command R+ with web search grounding
- `embed_texts` — Generate dense vector representations using Embed v3 (search/document)
- `rerank_documents` — High-precision neural reranking of candidate passages with Rerank 3
- `classify_text` — Multi-class intent classification and toxic content moderation
- `tokenize_text` — Inspect token boundaries and cost metrics

## Client Configuration
```json
{
  "mcpServers": {
    "cohere": {
      "command": "npx",
      "args": [
        "-y",
        "@cohere/mcp-server"
      ],
      "env": {
        "COHERE_API_KEY": "YOUR_COHERE_API_KEY"
      }
    }
  }
}
```

## Security & Best Practices
- Validate input batch sizes. Restrict reranking input length to max context window.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
