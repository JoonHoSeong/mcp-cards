# SerpApi Multi-Engine Search MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | SerpApi Multi-Engine Search MCP Server |
| **Category** | Search/Scraping |
| **Official** | ⭐ Official (SerpApi) |
| **Source** | https://github.com/serpapi/mcp-server |
| **Transport** | stdio |
| **Install** | `npx -y serpapi-mcp` |

## Tools & Capabilities
- `search_google` — Query Google Search with localization, pagination, and snippet extraction
- `search_google_scholar` — Query academic research publications, citations, and author metrics
- `search_google_news` — Retrieve real-time breaking news articles with timestamp sorting
- `search_bing` — Execute Bing Web and News search queries
- `search_naver` — Query Naver Search (Korean web, blog, and cafe results)

## Client Configuration
```json
{
  "mcpServers": {
    "serpapi": {
      "command": "npx",
      "args": [
        "-y",
        "serpapi-mcp"
      ],
      "env": {
        "SERPAPI_API_KEY": "YOUR_SERPAPI_KEY"
      }
    }
  }
}
```

## Security & Best Practices
- Enforce search query rate limiting to preserve API quota credits.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
