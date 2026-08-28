# Exa AI Neural Search MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | Exa AI Neural Search MCP Server |
| **Category** | Search/Neural |
| **Official** | ⭐ Official (Exa) |
| **Source** | https://github.com/exa-labs/exa-mcp-server |
| **Transport** | stdio |
| **Install** | `npx -y exa-mcp-server` |

## Tools & Capabilities
- `search` — Perform neural semantic search returning high-quality web links and summaries
- `find_similar` — Search the web for documents semantically similar to a target URL
- `get_contents` — Fetch clean, parsed HTML/markdown content from list of URL IDs
- `search_and_contents` — Execute search and extract full text contents in a single atomic call
- `filter_by_category` — Filter search results across research papers, news, company blogs, or code

## Client Configuration
```json
{
  "mcpServers": {
    "exa": {
      "command": "npx",
      "args": [
        "-y",
        "exa-mcp-server"
      ],
      "env": {
        "EXA_API_KEY": "YOUR_EXA_API_KEY"
      }
    }
  }
}
```

## Security & Best Practices
- Enforce maximum page extract limits to avoid large token context overflow.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
