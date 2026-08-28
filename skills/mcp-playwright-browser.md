# Playwright MCP Server

## Overview
| Field | Value |
|-------|-------|
| **Name** | Playwright MCP |
| **Category** | Testing/Automation |
| **Official** | Official (Microsoft) |
| **Source** | https://github.com/microsoft/playwright-mcp |
| **Transport** | stdio |
| **Install** | `npx -y @playwright/mcp` |

## Description
Playwright MCP server enables AI agents to automate browser interactions including navigation, form filling, clicking, and screenshot capture. Built by Microsoft, it provides full browser automation capabilities through the Model Context Protocol, supporting accessibility-first testing and end-to-end validation of web applications.

## Key Tools
| Tool | Description |
|------|-------------|
| `navigate` | Navigate to a URL and wait for page load |
| `click` | Click on elements identified by selector or accessibility role |
| `fill` | Fill form inputs with specified values |
| `screenshot` | Capture full-page or element screenshots |
| `get_text` | Extract text content from page elements |
| `evaluate` | Execute JavaScript in the browser context |
| `wait_for_selector` | Wait for an element to appear in the DOM |
| `get_accessibility_tree` | Get the accessibility tree for the page |

## Configuration
```json
{
  "mcpServers": {
    "playwright": {
      "command": "npx",
      "args": ["-y", "@playwright/mcp"],
      "env": {
        "PLAYWRIGHT_HEADLESS": "true"
      }
    }
  }
}
```

## Use Cases
1. End-to-end testing of web dashboards and APIs  2. Visual regression testing with screenshot comparisons  3. Automated form submission and data extraction

## NVIDIA Skill Combinations
| Purpose | NVIDIA Skill | Reason |
|---------|-------------|--------|
| Test video analytics endpoints | vss-setup-video-analytics-api | Validate VSS API responses and web UI interactions |
| Verify deployment dashboards | aiq-deploy | Test AIQ toolkit web interfaces after deployment |
| Test Omniverse web viewers | omniverse-realtime-viewer | Automate testing of Omniverse streaming web clients |

## Prerequisites
- Node.js 18+ installed
- Chromium browser (auto-installed by Playwright)

## Security Notes
- Browser runs in sandboxed context; restrict navigation to trusted domains
- Avoid filling real credentials in automated tests; use test accounts
- Screenshots may capture sensitive data; handle output securely

## References
- https://github.com/microsoft/playwright-mcp
- https://playwright.dev/docs/intro
