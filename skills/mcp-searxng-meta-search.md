# SearXNG Private Meta-Search MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | SearXNG Private Meta-Search MCP Server |
| **Category** | Search/Privacy |
| **Official** | ⭐ Verified (ihor-sokoliuk/mcp-searxng) |
| **Source** | https://github.com/ihor-sokoliuk/mcp-searxng |
| **Transport** | stdio |
| **Install** | `npx -y @ihor-sokoliuk/mcp-searxng` |

## Tools & Capabilities
- `meta_search` — Aggregate search results from 70+ search engines simultaneously with zero tracking
- `search_category` — Search specific domains (IT/Code, Science, Files, Images, Videos, News)
- `extract_clean_markdown` — Scrape and extract readable markdown from search result links

## Client Configuration
```json
{
  "mcpServers": {
    "searxng": {
      "command": "npx",
      "args": [
        "-y",
        "@ihor-sokoliuk/mcp-searxng"
      ],
      "env": {
        "SEARXNG_BASE_URL": "http://localhost:8080"
      }
    }
  }
}
```

## Security & Best Practices
- Self-hosted private meta-search engine integration. Zero outbound tracking or profiling.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
