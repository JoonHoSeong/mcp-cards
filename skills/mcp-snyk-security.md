# Snyk MCP Server

## Overview
| Field | Value |
|-------|-------|
| **Name** | saw-mcp |
| **Category** | Security |
| **Official** | Yes |
| **Source** | https://github.com/snyk/saw-mcp |
| **Transport** | stdio |
| **Install** | `npx @snyk/mcp-server` |

## Description
Official Snyk MCP server for scanning projects for vulnerabilities, managing security findings, and running DAST scans. Enables AI-assisted security workflows including dependency vulnerability detection, target onboarding, and remediation guidance across application codebases.

## Key Tools
| Tool | Description |
|------|-------------|
| `scan_project` | Scan a project for known vulnerabilities |
| `list_vulnerabilities` | List vulnerabilities for a scanned target |
| `get_vulnerability` | Get detailed vulnerability information |
| `list_targets` | List monitored targets/projects |
| `run_dast_scan` | Run dynamic application security testing |
| `configure_auth` | Configure Snyk authentication |
| `onboard_target` | Onboard a new project for monitoring |

## Configuration
```json
{
  "mcpServers": {
    "snyk": {
      "command": "npx",
      "args": ["@snyk/mcp-server"],
      "env": {
        "SNYK_TOKEN": "your-api-token"
      }
    }
  }
}
```

## Use Cases
1. Scan DPU firmware and driver code for known vulnerability patterns
2. Monitor dependency vulnerabilities in CUDA and inference service projects
3. Run DAST scans against deployed AI API endpoints for security validation

## NVIDIA Skill Combinations
| Purpose | NVIDIA Skill | Reason |
|---------|-------------|--------|
| Scan DPU code | doca-setup | Scan DOCA DPU application code for security vulnerabilities |

## Prerequisites
- Snyk account (free tier available)
- `SNYK_TOKEN` — API token from Snyk account settings

## Security Notes
- Use organization-scoped service account tokens for CI/CD integrations
- Avoid using personal tokens in shared environments; rotate regularly

## References
- https://github.com/snyk/saw-mcp
- https://docs.snyk.io/snyk-api
