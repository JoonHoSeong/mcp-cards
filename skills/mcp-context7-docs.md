# Context7 MCP Server

## Overview
| Field | Value |
|-------|-------|
| **Name** | Context7 MCP Server |
| **Category** | Development/Documentation |
| **Official** | ⭐ Official (TrueFoundry) |
| **Source** | https://github.com/truefoundry/context7-mcp-server |
| **Transport** | stdio |
| **Install** | `npx -y @truefoundry/context7-mcp` |

## Description
MCP server that provides up-to-date library documentation directly in the IDE. Prevents hallucinated API usage by fetching real, version-specific documentation and code examples. Supports thousands of libraries and frameworks, making it an essential companion for any development workflow that relies on accurate API references.

## Key Tools
| Tool | Description |
|------|-------------|
| `resolve_library` | Resolve a library name to its Context7 identifier |
| `get_library_docs` | Fetch documentation for a specific library and version |
| `search_docs` | Search across library documentation for specific topics |
| `get_code_examples` | Get working code examples for a library feature |

## Configuration
```json
{
  "mcpServers": {
    "context7": {
      "command": "npx",
      "args": ["-y", "@truefoundry/context7-mcp"]
    }
  }
}
```

## Use Cases
1. Get up-to-date library documentation directly in IDE without browser switching
2. Prevent hallucinated API usage by grounding responses in real documentation
3. Retrieve version-specific code examples for exact library versions in use
4. Library migration guidance — compare APIs across versions
5. Discover new features and deprecated methods in latest releases

## NVIDIA Skill Combinations
| Purpose | NVIDIA Skill | Reason |
|---------|-------------|--------|
| Latest SDK documentation | ALL NVIDIA skills | Provides current docs for NeMo, TAO, DOCA, TensorRT, and other NVIDIA SDKs to ensure correct API usage |
| Reference docs for card creation | `skill-card-generator` | Fetch accurate library documentation when generating new skill cards |

## Prerequisites
- None — Context7 is a free service with no authentication required
- Node.js 18+ for npx installation
- Internet connection for fetching documentation

## Security Notes
- Read-only service — no credentials needed
- No data is sent to Context7 beyond the library query
- No API keys or tokens required
- Safe for use in any environment including air-gapped setups (with internet proxy)

## References
- [GitHub Repository](https://github.com/truefoundry/context7-mcp-server)
- [Context7 Website](https://context7.com/)
- [TrueFoundry](https://www.truefoundry.com/)
