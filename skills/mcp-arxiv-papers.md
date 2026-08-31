# arXiv Academic Papers MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | arXiv Academic Papers MCP Server |
| **Category** | Academic/Research |
| **Official** | ⭐ Open Source Verified |
| **Source** | https://github.com/arxiv-mcp/arxiv-mcp-server |
| **Transport** | stdio |
| **Install** | `npx -y @arxiv-mcp/server` |

## Tools & Capabilities
- `search_papers` — Query arXiv papers by title, abstract, authors, category (cs.AI, cs.LG), and date
- `get_paper_details` — Retrieve full metadata, authors, DOI, and paper summary
- `download_and_extract_pdf` — Download PDF, parse sections, equations, and tables into structured markdown
- `list_latest_by_category` — Stream newest preprint releases in specified AI/physics categories

## Client Configuration
```json
{
  "mcpServers": {
    "arxiv": {
      "command": "npx",
      "args": [
        "-y",
        "@arxiv-mcp/server"
      ]
    }
  }
}
```

## Security & Best Practices
- Rate-limited polite scraping complying with arXiv API access guidelines.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
