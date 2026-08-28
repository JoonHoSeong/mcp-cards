# Make (Integromat) MCP Server

## Overview
| Field | Value |
|-------|-------|
| **Name** | Make MCP |
| **Category** | Automation |
| **Official** | Yes |
| **Source** | https://github.com/integromat/make-mcp-server |
| **Transport** | stdio |
| **Install** | `npx @integromat/mcp-server` |

## Description
The Make MCP server enables AI agents to interact with Make's visual automation platform. It provides access to scenarios, webhooks, and execution history, allowing agents to trigger complex multi-step workflows, monitor execution status, and integrate with hundreds of connected services.

## Key Tools
| Tool | Description |
|------|-------------|
| `list_scenarios` | List available automation scenarios |
| `run_scenario` | Trigger a scenario execution |
| `get_scenario_status` | Check scenario run status |
| `list_connections` | List configured app connections |
| `create_webhook` | Create a webhook trigger endpoint |
| `get_execution_history` | Retrieve past execution logs |

## Configuration
```json
{
  "mcpServers": {
    "make": {
      "command": "npx",
      "args": ["@integromat/mcp-server"],
      "env": {
        "MAKE_API_TOKEN": "${MAKE_API_TOKEN}"
      }
    }
  }
}
```

## Use Cases
1. AI-triggered multi-step automation workflows
2. Webhook-driven event processing pipelines
3. Monitoring and debugging automation execution history

## NVIDIA Skill Combinations
| Purpose | NVIDIA Skill | Reason |
|---------|-------------|--------|
| Trigger training workflows | tao-launch-workflow | Orchestrate TAO training jobs via Make scenarios as automation bridges |

## Prerequisites
- `MAKE_API_TOKEN` — Make API token with scenario and webhook permissions

## Security Notes
- API token grants ability to execute scenarios that may interact with external services
- Webhook endpoints should be secured with authentication to prevent unauthorized triggers

## References
- https://github.com/integromat/make-mcp-server
- https://www.make.com/en/api-documentation
