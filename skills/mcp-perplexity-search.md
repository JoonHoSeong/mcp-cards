# Perplexity Search MCP Server

## Overview
| Field | Value |
|-------|-------|
| **Name** | perplexity-mcp |
| **Category** | AI/ML |
| **Official** | Yes |
| **Source** | https://github.com/perplexityai/modelcontextprotocol |
| **Transport** | stdio |
| **Install** | `npx @perplexity/mcp-server` |

## Description
Perplexity MCP Server brings real-time web search and deep research capabilities to AI agents. It provides access to Perplexity's search engine with source citations, enabling LLM workflows to retrieve up-to-date information, conduct multi-step research, and verify facts with traceable references — all integrated directly into the development environment.

## Key Tools
| Tool | Description |
|------|-------------|
| `search` | Perform a web search with AI-summarized results |
| `research` | Conduct deep multi-step research on a topic |
| `get_citations` | Retrieve source citations for a previous search |
| `ask_followup` | Ask a follow-up question in a research session |
| `search_with_sources` | Search and return results with inline source links |
| `deep_research` | Extended research with comprehensive source analysis |

## Configuration
```json
{
  "mcpServers": {
    "perplexity": {
      "command": "npx",
      "args": ["@perplexity/mcp-server"],
      "env": {
        "PERPLEXITY_API_KEY": "pplx-your_key_here"
      }
    }
  }
}
```

## Use Cases
1. Research latest ML techniques, papers, and benchmarks during model development
2. Verify API documentation and library compatibility with real-time web search
3. Conduct deep research on domain-specific topics for RAG knowledge base curation

## NVIDIA Skill Combinations
| Purpose | NVIDIA Skill | Reason |
|---------|-------------|--------|
| Augment deep research | aiq-research | Combine Perplexity web search with AIQ structured research workflows |
| Discover relevant skills | nvidia-skill-finder | Search for NVIDIA skills matching specific use-case requirements |

## Prerequisites
- `PERPLEXITY_API_KEY` from https://www.perplexity.ai/settings/api
- Node.js 18+ (for npx)

## Security Notes
- API key usage is metered; set billing limits to prevent unexpected charges
- Search queries may be logged by Perplexity; avoid searching sensitive internal information

## References
- https://github.com/perplexityai/modelcontextprotocol
- https://docs.perplexity.ai/
