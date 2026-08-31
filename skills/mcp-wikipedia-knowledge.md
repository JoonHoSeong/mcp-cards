# Wikipedia Knowledge & Reference MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | Wikipedia Knowledge & Reference MCP Server |
| **Category** | Knowledge/Search |
| **Official** | ⭐ Official (Wikimedia) |
| **Source** | https://github.com/wikimedia/mcp-wikipedia |
| **Transport** | stdio |
| **Install** | `npx -y @wikimedia/mcp-server` |

## Tools & Capabilities
- `search_articles` — Query Wikipedia articles across all languages by keyword and semantic similarity
- `get_article_summary` — Retrieve introductory abstract and infobox facts
- `get_article_section` — Fetch specific section text and structured table data without full page overhead
- `get_page_references` — Parse primary external citations and bibliography links

## Client Configuration
```json
{
  "mcpServers": {
    "wikipedia": {
      "command": "npx",
      "args": [
        "-y",
        "@wikimedia/mcp-server"
      ]
    }
  }
}
```

## Security & Best Practices
- Pure read-only Wikimedia REST API integration. Custom User-Agent header compliance.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
