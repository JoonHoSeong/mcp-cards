# Pipedream MCP Server

## Overview
| Field | Value |
|-------|-------|
| **Name** | Pipedream MCP |
| **Category** | Automation |
| **Official** | Yes |
| **Source** | https://github.com/PipedreamHQ/pipedream/tree/master/modelcontextprotocol |
| **Transport** | stdio |
| **Install** | `npx @pipedream/mcp-server` |

## Description
The Pipedream MCP server connects AI agents to Pipedream's event-driven automation platform with access to 2,500+ APIs and 8,000+ pre-built tools. It enables workflow deployment, event source management, and triggered execution, making it ideal for building event-driven AI pipelines.

## Key Tools
| Tool | Description |
|------|-------------|
| `list_workflows` | List deployed workflows |
| `trigger_workflow` | Trigger a workflow execution |
| `list_sources` | List event sources |
| `get_events` | Retrieve events from a source |
| `deploy_workflow` | Deploy a new workflow |
| `list_connected_accounts` | List connected API accounts |

## Configuration
```json
{
  "mcpServers": {
    "pipedream": {
      "command": "npx",
      "args": ["@pipedream/mcp-server"],
      "env": {
        "PIPEDREAM_API_KEY": "${PIPEDREAM_API_KEY}"
      }
    }
  }
}
```

## Use Cases
1. Event-driven AI workflow automation across 2,500+ APIs
2. Real-time event processing and webhook management
3. Deploying and managing serverless automation pipelines

## NVIDIA Skill Combinations
| Purpose | NVIDIA Skill | Reason |
|---------|-------------|--------|
| Event-driven training | tao-launch-workflow | Trigger TAO training workflows based on real-time data events |

## Prerequisites
- `PIPEDREAM_API_KEY` — Pipedream API key with workflow and source permissions

## Security Notes
- API key provides access to deploy and execute workflows; store securely
- Connected accounts inherit their individual service permissions

## References
- https://github.com/PipedreamHQ/pipedream/tree/master/modelcontextprotocol
- https://pipedream.com/docs/api
