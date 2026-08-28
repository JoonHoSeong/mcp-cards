# NuGet MCP Server

## Overview
| Field | Value |
|-------|-------|
| **Name** | NuGet MCP |
| **Category** | Package Management |
| **Official** | Official (Microsoft) |
| **Source** | https://github.com/microsoft/nuget-mcp |
| **Transport** | stdio |
| **Install** | Built-in with Visual Studio 2026+ |

## Description
NuGet MCP server enables AI agents to search, inspect, and analyze .NET packages from the NuGet registry. It provides dependency resolution, version comparison, and vulnerability scanning capabilities, making it essential for .NET development workflows that require package management decisions.

## Key Tools
| Tool | Description |
|------|-------------|
| `search_packages` | Search NuGet gallery by name, tags, or description |
| `get_package_info` | Get detailed package metadata including authors and license |
| `list_versions` | List all available versions of a package |
| `get_dependencies` | Resolve dependency tree for a package version |
| `get_vulnerabilities` | Check known vulnerabilities for a package version |

## Configuration
```json
{
  "mcpServers": {
    "nuget": {
      "command": "dotnet",
      "args": ["mcp", "nuget"],
      "env": {
        "NUGET_SOURCE": "https://api.nuget.org/v3/index.json"
      }
    }
  }
}
```

## Use Cases
1. Find compatible packages for .NET Holoscan application development  2. Audit dependency vulnerabilities before production deployment  3. Compare package versions and breaking changes during upgrades

## NVIDIA Skill Combinations
| Purpose | NVIDIA Skill | Reason |
|---------|-------------|--------|
| Holoscan .NET packages | hsb-app | Find and validate NuGet packages for Holoscan Sensor Bridge .NET applications |
| CUDA .NET bindings | jetson-init-target | Locate CUDA-related NuGet packages for Jetson .NET development |

## Prerequisites
- Visual Studio 2026+ (built-in) or .NET SDK 9+
- NuGet source configured (defaults to nuget.org)

## Security Notes
- Vulnerability scanning should be run before adding new dependencies
- Private NuGet feeds may require authentication tokens
- Verify package publisher identity to avoid typosquatting attacks

## References
- https://github.com/microsoft/nuget-mcp
- https://learn.microsoft.com/nuget/
- https://www.nuget.org/
