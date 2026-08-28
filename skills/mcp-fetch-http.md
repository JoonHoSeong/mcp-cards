# Anthropic Official Fetch MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | Anthropic Official Fetch MCP Server |
| **Category** | Core/Web |
| **Official** | ⭐ Official (Anthropic Reference) |
| **Source** | https://github.com/modelcontextprotocol/servers/tree/main/src/fetch |
| **Transport** | stdio |
| **Install** | `npx -y @modelcontextprotocol/server-fetch` |

## Tools & Capabilities
- `fetch` — Fetch content from URL via HTTP GET with automatic HTML-to-Markdown conversion
- `fetch_raw` — Retrieve raw response body, status code, and response headers
- `fetch_json` — Request and parse JSON API endpoint responses directly
- `set_user_agent` — Customize request User-Agent header for specialized scraping
- `inspect_headers` — Inspect HTTP response headers, CORS policies, and caching directives

## Client Configuration
```json
{
  "mcpServers": {
    "fetch": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-fetch"
      ]
    }
  }
}
```

## Security & Best Practices
- Block internal loopback (`127.0.0.1`, `localhost`) and cloud metadata endpoints (`169.254.169.254`).

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
