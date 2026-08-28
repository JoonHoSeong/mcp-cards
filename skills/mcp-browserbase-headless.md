# Browserbase Headless Browser MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | Browserbase Headless Browser MCP Server |
| **Category** | Automation/Browser |
| **Official** | ⭐ Official (Browserbase) |
| **Source** | https://github.com/browserbase/mcp-server-browserbase |
| **Transport** | stdio |
| **Install** | `npx -y @browserbasehq/mcp-browserbase` |

## Tools & Capabilities
- `create_session` — Launch managed cloud Chromium browser session with proxy & stealth modes
- `navigate` — Open URL and wait for network idle or selector visibility
- `click_element` — Click DOM element identified by selector or natural coordinates
- `take_screenshot` — Capture full-page or element screenshot and return base64/URL
- `inspect_console_logs` — Retrieve browser console errors, warnings, and network failures

## Client Configuration
```json
{
  "mcpServers": {
    "browserbase": {
      "command": "npx",
      "args": [
        "-y",
        "@browserbasehq/mcp-browserbase"
      ],
      "env": {
        "BROWSERBASE_API_KEY": "YOUR_BROWSERBASE_KEY",
        "BROWSERBASE_PROJECT_ID": "YOUR_PROJECT_ID"
      }
    }
  }
}
```

## Security & Best Practices
- Isolated cloud browser sandbox. Automatic session cleanup and proxy rotation.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
