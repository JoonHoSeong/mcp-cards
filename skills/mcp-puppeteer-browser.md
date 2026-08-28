# Puppeteer MCP Server

## Overview
| Field | Value |
|-------|-------|
| **Name** | Puppeteer MCP |
| **Category** | Browser Automation |
| **Official** | Reference Implementation |
| **Source** | https://github.com/modelcontextprotocol/servers/tree/main/src/puppeteer |
| **Transport** | stdio |
| **Install** | `npx @modelcontextprotocol/server-puppeteer` |

## Description
The Puppeteer MCP server gives AI agents full browser automation capabilities via headless Chrome. It supports navigation, screenshots, DOM interaction, JavaScript evaluation, and PDF generation, enabling web scraping, testing, and visual content extraction workflows.

## Key Tools
| Tool | Description |
|------|-------------|
| `navigate` | Navigate to a URL |
| `screenshot` | Capture page or element screenshot |
| `click` | Click an element on the page |
| `type` | Type text into an input field |
| `evaluate` | Execute JavaScript in page context |
| `pdf` | Generate PDF from page content |
| `get_content` | Get page HTML or text content |
| `wait_for` | Wait for selector or navigation event |

## Configuration
```json
{
  "mcpServers": {
    "puppeteer": {
      "command": "npx",
      "args": ["@modelcontextprotocol/server-puppeteer"],
      "env": {
        "PUPPETEER_EXECUTABLE_PATH": "/usr/bin/chromium"
      }
    }
  }
}
```

## Use Cases
1. Web scraping and structured data extraction
2. Automated UI testing and visual regression detection
3. Dynamic content capture from JavaScript-rendered pages

## NVIDIA Skill Combinations
| Purpose | NVIDIA Skill | Reason |
|---------|-------------|--------|
| Scrape video feeds | vss-setup-video-analytics-api | Extract video stream URLs and metadata for VSS pipeline ingestion |

## Prerequisites
- Node.js runtime
- Chrome or Chromium browser installed

## Security Notes
- Browser automation can interact with authenticated sessions; isolate execution context
- Evaluating arbitrary JavaScript in page context carries injection risks

## References
- https://github.com/modelcontextprotocol/servers/tree/main/src/puppeteer
- https://pptr.dev/
