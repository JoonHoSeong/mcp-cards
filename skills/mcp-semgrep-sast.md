# Semgrep MCP Server

## Overview
| Field | Value |
|-------|-------|
| **Name** | semgrep-mcp |
| **Category** | Security/SAST |
| **Official** | Yes |
| **Source** | https://github.com/semgrep/mcp |
| **Transport** | stdio |
| **Install** | `npx @semgrep/mcp` |

## Description
Official Semgrep MCP server for static application security testing (SAST) using pattern-based code analysis. Enables AI-assisted code scanning with customizable rules, severity filtering, and directory-level analysis for detecting security anti-patterns and bugs in source code.

## Key Tools
| Tool | Description |
|------|-------------|
| `scan_code` | Scan a code snippet against Semgrep rules |
| `list_rules` | List available scanning rules and rulesets |
| `get_rule` | Get details for a specific rule |
| `scan_directory` | Scan an entire directory recursively |
| `get_findings` | Get findings from a completed scan |
| `filter_by_severity` | Filter scan results by severity level |

## Configuration
```json
{
  "mcpServers": {
    "semgrep": {
      "command": "npx",
      "args": ["@semgrep/mcp"],
      "env": {
        "SEMGREP_APP_TOKEN": "optional-for-pro-rules"
      }
    }
  }
}
```

## Use Cases
1. Scan DeepStream pipeline code for memory safety and resource leak patterns
2. Enforce secure coding rules across CUDA and Python inference codebases
3. Filter high-severity findings to prioritize critical security fixes before deployment

## NVIDIA Skill Combinations
| Purpose | NVIDIA Skill | Reason |
|---------|-------------|--------|
| Scan pipeline code | deepstream-dev | Scan DeepStream pipeline code for security and correctness issues |

## Prerequisites
- None for open source rules (community rulesets included)
- `SEMGREP_APP_TOKEN` — Optional, required for Semgrep Pro rules and cloud findings

## Security Notes
- Open source rules run locally with no data sent externally
- Pro rules with SEMGREP_APP_TOKEN may transmit findings metadata to Semgrep Cloud

## References
- https://github.com/semgrep/mcp
- https://semgrep.dev/docs/
