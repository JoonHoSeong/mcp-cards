# Brave Search MCP Server

## Overview
| Field | Value |
|-------|-------|
| **Name** | Brave Search MCP |
| **Category** | Web Search |
| **Official** | Reference Implementation |
| **Source** | https://github.com/modelcontextprotocol/servers/tree/main/src/brave-search |
| **Transport** | stdio |
| **Install** | `npx @modelcontextprotocol/server-brave-search` |

## Description
The Brave Search MCP server provides AI agents with web, news, local, and image search capabilities powered by Brave's independent search index. It enables grounding agent responses in current web data without relying on major search engine ecosystems.

## Key Tools
| Tool | Description |
|------|-------------|
| `web_search` | General web search with ranked results |
| `local_search` | Search for local businesses and places |
| `news_search` | Search recent news articles |
| `image_search` | Search for images by query |

## Configuration
```json
{
  "mcpServers": {
    "brave-search": {
      "command": "npx",
      "args": ["@modelcontextprotocol/server-brave-search"],
      "env": {
        "BRAVE_API_KEY": "${BRAVE_API_KEY}"
      }
    }
  }
}
```

## Use Cases
1. Grounding AI responses with current web information
2. Real-time news monitoring and research
3. Local business discovery and verification

## NVIDIA Skill Combinations
| Purpose | NVIDIA Skill | Reason |
|---------|-------------|--------|
| Ground answers in web | aiq-research | Combine web search results with deep research for comprehensive answers |

## Prerequisites
- `BRAVE_API_KEY` — Brave Search API key (free tier available)

## Security Notes
- Search queries may reveal user intent; consider privacy implications of logged queries
- API key has rate limits; monitor usage to avoid service disruption

## References
- https://github.com/modelcontextprotocol/servers/tree/main/src/brave-search
- https://brave.com/search/api/
