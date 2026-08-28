# Stainless MCP Generator

## Overview
| Field | Value |
|-------|-------|
| **Name** | Stainless MCP Generator |
| **Category** | Code Generation |
| **Official** | Tool |
| **Source** | https://stainless.com |
| **Transport** | stdio |
| **Install** | `npx -y @stainless/mcp-generator` |

## Description
Stainless MCP Generator automatically creates MCP server implementations from OpenAPI specifications. It transforms any REST API into a fully typed MCP tool server with proper input validation, error handling, and documentation, enabling rapid integration of existing APIs into AI agent workflows without manual tool authoring.

## Key Tools
| Tool | Description |
|------|-------------|
| `generate_mcp_from_openapi` | Generate a complete MCP server from an OpenAPI spec |
| `validate_spec` | Validate an OpenAPI specification for MCP compatibility |
| `list_endpoints` | List API endpoints that will become MCP tools |
| `generate_sdk` | Generate typed SDK clients alongside the MCP server |

## Configuration
```json
{
  "mcpServers": {
    "stainless": {
      "command": "npx",
      "args": ["-y", "@stainless/mcp-generator"],
      "env": {
        "OPENAPI_SPEC_PATH": "/path/to/openapi.yaml"
      }
    }
  }
}
```

## Use Cases
1. Generate MCP servers for NVIDIA NGC API endpoints  2. Create typed tool interfaces from any REST API specification  3. Rapidly prototype AI agent integrations with existing services

## NVIDIA Skill Combinations
| Purpose | NVIDIA Skill | Reason |
|---------|-------------|--------|
| MCP for any NVIDIA API | ALL | Generate MCP tool servers from any NVIDIA service OpenAPI spec |
| NIM API tools | nim-agent-blueprint | Create MCP tools from NIM inference API specifications |
| Omniverse API tools | omniverse-realtime-viewer | Generate MCP server from Omniverse Kit REST API specs |

## Prerequisites
- Valid OpenAPI 3.0+ specification (YAML or JSON)
- Node.js 18+ for generated server runtime

## Security Notes
- Generated servers inherit security models from the source API spec
- Review generated authentication handling before production deployment
- API keys in specs should be parameterized, not hardcoded in generated code

## References
- https://stainless.com
- https://docs.stainless.com/mcp
- https://swagger.io/specification/
