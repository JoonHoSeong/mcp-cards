# Tavily AI Search MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | Tavily AI Search MCP Server |
| **Category** | Search/Research |
| **Official** | ⭐ Official (Tavily) |
| **Source** | https://github.com/tavily-ai/tavily-mcp |
| **Transport** | stdio |
| **Install** | `npx -y @tavily/mcp-server` |

## Tools & Capabilities
- `tavily_search` — Execute AI-optimized web search returning clean content snippets and raw text
- `tavily_qna_search` — Get direct synthesized factual answers to natural language questions
- `tavily_extract` — Extract parsed markdown content directly from specific URL targets
- `tavily_context_search` — Retrieve search results optimized for RAG context window packing
- `filter_search_domains` — Restrict search queries to specific trusted domain allowlists

## Client Configuration
```json
{
  "mcpServers": {
    "tavily": {
      "command": "npx",
      "args": [
        "-y",
        "@tavily/mcp-server"
      ],
      "env": {
        "TAVILY_API_KEY": "YOUR_TAVILY_API_KEY"
      }
    }
  }
}
```

## Security & Best Practices
- Use domain filtering to prevent scraping internal networks. Do not pass credentials in search queries.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
