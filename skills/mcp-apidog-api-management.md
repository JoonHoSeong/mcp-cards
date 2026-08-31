# Apidog API Platform MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | Apidog API Platform MCP Server |
| **Category** | Developer/API |
| **Official** | ⭐ Official (Apidog) |
| **Source** | https://github.com/apidog/mcp-server-apidog |
| **Transport** | stdio |
| **Install** | `npx -y @apidog/mcp-server` |

## Tools & Capabilities
- `get_api_spec` — Retrieve endpoint schemas, request/response bodies, and status codes from Apidog project
- `create_api_endpoint` — Generate new API specification directly from natural language or code models
- `run_test_scenario` — Trigger automated API test cases and assert response validations
- `sync_mock_data` — Generate intelligent mock server response payloads for frontend development
- `export_openapi_json` — Export complete OpenAPI 3.0 / Swagger specifications

## Client Configuration
```json
{
  "mcpServers": {
    "apidog": {
      "command": "npx",
      "args": [
        "-y",
        "@apidog/mcp-server"
      ],
      "env": {
        "APIDOG_ACCESS_TOKEN": "YOUR_APIDOG_TOKEN",
        "APIDOG_PROJECT_ID": "123456"
      }
    }
  }
}
```

## Security & Best Practices
- Personal Access Token authentication with project-level RBAC isolation.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
