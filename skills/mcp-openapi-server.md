# OpenAPI MCP Server

## Overview
| Field | Value |
|-------|-------|
| **Name** | OpenAPI MCP Server |
| **Category** | API Integration |
| **Official** | Community |
| **Source** | https://github.com/ivo-toby/mcp-openapi-server |
| **Transport** | stdio |
| **Install** | `npx -y mcp-openapi-server` |

## Description
OpenAPI MCP Server dynamically exposes any REST API as MCP tools by reading its OpenAPI specification at runtime. Unlike code generators, it provides a live proxy that translates MCP tool calls into HTTP requests, enabling AI agents to interact with any documented REST API without pre-generated code or custom server development.

## Key Tools
| Tool | Description |
|------|-------------|
| `list_endpoints` | Discover available API endpoints from the loaded spec |
| `call_endpoint` | Execute an API call with parameters and authentication |
| `get_schema` | Retrieve request/response schemas for an endpoint |
| `discover_apis` | Find and load OpenAPI specs from a URL or directory |

## Configuration
```json
{
  "mcpServers": {
    "openapi": {
      "command": "npx",
      "args": ["-y", "mcp-openapi-server"],
      "env": {
        "OPENAPI_SPEC_URL": "https://api.example.com/openapi.json",
        "API_AUTH_HEADER": "Authorization: Bearer <token>"
      }
    }
  }
}
```

## Use Cases
1. Instantly expose any documented API as AI agent tools  2. Prototype integrations without writing custom MCP servers  3. Aggregate multiple APIs into a unified tool surface for agents

## NVIDIA Skill Combinations
| Purpose | NVIDIA Skill | Reason |
|---------|-------------|--------|
| Expose NVIDIA APIs as tools | ALL | Dynamically proxy any NVIDIA REST API as MCP tools without code generation |
| NIM endpoint access | nim-agent-blueprint | Live proxy to NIM inference endpoints for agent orchestration |
| NGC catalog access | nemotron-customize | Browse and interact with NGC model catalog APIs |

## Prerequisites
- Valid OpenAPI specification URL or local file path
- API authentication credentials (if required by target API)

## Security Notes
- All API calls pass through the proxy; ensure HTTPS for sensitive endpoints
- Authentication headers are stored in environment variables; never log them
- Validate OpenAPI specs from untrusted sources before loading

## References
- https://github.com/ivo-toby/mcp-openapi-server
- https://spec.openapis.org/oas/v3.1.0
