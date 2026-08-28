# Microsoft Learn MCP Server

## Overview
| Field | Value |
|-------|-------|
| **Name** | MS Learn MCP |
| **Category** | Documentation |
| **Official** | Official (Microsoft) |
| **Source** | https://github.com/microsoftdocs/mcp |
| **Transport** | stdio |
| **Install** | `npx -y @microsoft/docs-mcp` |

## Description
Microsoft Learn MCP server provides AI agents with searchable access to Microsoft's technical documentation, code samples, and product guides. It enables context-aware documentation retrieval for Azure, .NET, Windows, and other Microsoft technologies, supporting developers who need authoritative reference material during coding sessions.

## Key Tools
| Tool | Description |
|------|-------------|
| `search_docs` | Full-text search across Microsoft Learn documentation |
| `get_doc_page` | Retrieve a specific documentation page by path or URL |
| `list_products` | List available product documentation categories |
| `get_code_samples` | Find and retrieve code samples for a technology |

## Configuration
```json
{
  "mcpServers": {
    "microsoft-learn": {
      "command": "npx",
      "args": ["-y", "@microsoft/docs-mcp"]
    }
  }
}
```

## Use Cases
1. Reference Azure documentation during cloud infrastructure development  2. Find .NET code samples for Holoscan application development  3. Look up Windows driver documentation for DPU integration

## NVIDIA Skill Combinations
| Purpose | NVIDIA Skill | Reason |
|---------|-------------|--------|
| DPU development reference | doca-programming-guide | Cross-reference MS docs for Windows/Azure integration with DOCA SmartNIC programming |
| .NET Holoscan apps | hsb-app | Reference .NET documentation when building Holoscan Sensor Bridge applications |
| Azure GPU deployment | aiq-deploy | Look up Azure GPU VM documentation for NVIDIA AI deployment |

## Prerequisites
- None (public documentation, no authentication required)

## Security Notes
- Read-only access to public documentation; no sensitive data exposure
- No API keys required; safe for unrestricted use
- Content is cached; verify against live docs for time-sensitive information

## References
- https://github.com/microsoftdocs/mcp
- https://learn.microsoft.com/
