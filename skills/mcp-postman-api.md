# Postman MCP Server

## Overview
| Field | Value |
|-------|-------|
| **Name** | Postman MCP |
| **Category** | API Development |
| **Official** | Official |
| **Source** | https://github.com/postmanlabs/postman-mcp-server |
| **Transport** | stdio |
| **Install** | `npx -y @postman/mcp-server` |

## Description
Postman MCP server enables AI agents to interact with Postman workspaces, collections, and environments for API testing and development. It provides tools to run API requests, validate specifications, search across API collections, and manage testing environments programmatically through the Model Context Protocol.

## Key Tools
| Tool | Description |
|------|-------------|
| `list_collections` | List all collections in a workspace |
| `get_collection` | Get detailed collection with requests and examples |
| `run_request` | Execute an API request from a collection |
| `list_environments` | List available environments and variables |
| `get_api_schema` | Retrieve OpenAPI/Swagger schema for an API |
| `search_apis` | Search across workspace APIs by name or description |
| `validate_spec` | Validate an API specification against standards |

## Configuration
```json
{
  "mcpServers": {
    "postman": {
      "command": "npx",
      "args": ["-y", "@postman/mcp-server"],
      "env": {
        "POSTMAN_API_KEY": "<your-api-key>"
      }
    }
  }
}
```

## Use Cases
1. Automated API testing and validation in CI/CD  2. API specification linting and standards compliance  3. Environment-aware request execution for staging vs production

## NVIDIA Skill Combinations
| Purpose | NVIDIA Skill | Reason |
|---------|-------------|--------|
| Test AI APIs | aiq-deploy | Validate deployed AIQ toolkit endpoints against Postman collections |
| Validate NIM endpoints | nim-agent-blueprint | Test NVIDIA Inference Microservice API contracts |
| Verify video APIs | vss-setup-video-analytics-api | Run Postman collections against VSS API endpoints |

## Prerequisites
- `POSTMAN_API_KEY` from Postman account settings
- Postman workspace with collections configured

## Security Notes
- API key grants access to all workspace data; use workspace-scoped keys
- Request execution may hit live endpoints; use environment variables for staging
- Collection variables may contain secrets; avoid logging response bodies

## References
- https://github.com/postmanlabs/postman-mcp-server
- https://learning.postman.com/docs/developer/postman-api/
