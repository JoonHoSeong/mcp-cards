# DuckDuckGo Privacy Search MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | DuckDuckGo Privacy Search MCP Server |
| **Category** | Search/Web |
| **Official** | ⭐ Open Source Verified |
| **Source** | https://github.com/duckduckgo-mcp/mcp-server-ddg |
| **Transport** | stdio |
| **Install** | `npx -y @duckduckgo-mcp/server` |

## Tools & Capabilities
- `search_web` — Execute web searches with privacy protection and clean URL extraction
- `search_news` — Retrieve recent news headlines, sources, and publication timestamps
- `get_instant_answers` — Fetch Wikipedia/ZeroClick instant answers and calculator facts

## Client Configuration
```json
{
  "mcpServers": {
    "duckduckgo": {
      "command": "npx",
      "args": [
        "-y",
        "@duckduckgo-mcp/server"
      ]
    }
  }
}
```

## Security & Best Practices
- Zero tracking and zero API key requirement. Rate limits queries to prevent IP blocks.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
