# Zapier MCP Server

## Overview
| Field | Value |
|-------|-------|
| **Name** | Zapier MCP |
| **Category** | Automation |
| **Official** | Yes (hosted remote) |
| **Source** | https://zapier.com/mcp |
| **Transport** | Remote (HTTP + SSE, OAuth) |
| **Install** | Remote server — no local install required |

## Description
The Zapier MCP server provides AI agents with access to 8,000+ apps and 30,000+ actions through a single endpoint. Tools are dynamically generated based on connected apps, making it a universal automation layer that can trigger workflows across virtually any SaaS platform.

## Key Tools
| Tool | Description |
|------|-------------|
| *Dynamic* | Tools are generated based on connected apps |
| *Actions* | Access to 30,000+ actions across 8,000+ apps |
| *Triggers* | Initiate workflows from AI agent context |
| *Searches* | Find records across connected platforms |

## Configuration
```json
{
  "mcpServers": {
    "zapier": {
      "url": "https://zapier.com/mcp"
    }
  }
}
```

## Use Cases
1. Universal automation layer connecting AI agents to any SaaS app
2. Multi-step workflow orchestration across platforms
3. Event-driven automation triggered by AI decisions

## NVIDIA Skill Combinations
| Purpose | NVIDIA Skill | Reason |
|---------|-------------|--------|
| Universal automation layer | ALL NVIDIA Skills | Zapier bridges any NVIDIA skill output to 8,000+ downstream apps |

## Prerequisites
- Zapier account with OAuth authorization and connected apps configured

## Security Notes
- Connected apps inherit their individual permission scopes; audit each connection
- Dynamic tool generation means available actions change with account configuration

## References
- https://zapier.com/mcp
- https://zapier.com/developer
