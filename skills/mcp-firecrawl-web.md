# Firecrawl Web Scraping MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | Firecrawl Web Scraping MCP Server |
| **Category** | Search/Scraping |
| **Official** | ⭐ Official (Mendable / Firecrawl) |
| **Source** | https://github.com/mendableai/firecrawl-mcp-server |
| **Transport** | stdio |
| **Install** | `npx -y firecrawl-mcp` |

## Tools & Capabilities
- `scrape_url` — Scrape dynamic JS-rendered webpage and return clean GitHub-flavored markdown
- `crawl_site` — Crawl entire domain subpages with depth and path filter controls
- `map_site` — Fast site mapping to retrieve all discoverable URLs on a target domain
- `extract_structured_data` — Extract structured JSON data using LLM schema extraction prompts
- `get_crawl_status` — Check progress and results of background site crawl jobs

## Client Configuration
```json
{
  "mcpServers": {
    "firecrawl": {
      "command": "npx",
      "args": [
        "-y",
        "firecrawl-mcp"
      ],
      "env": {
        "FIRECRAWL_API_KEY": "YOUR_FIRECRAWL_KEY"
      }
    }
  }
}
```

## Security & Best Practices
- Respect robots.txt policies. Block crawling private/internal intranet IP addresses.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
