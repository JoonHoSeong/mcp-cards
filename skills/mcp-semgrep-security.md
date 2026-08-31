# Semgrep Code Security & SAST MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | Semgrep Code Security & SAST MCP Server |
| **Category** | Security/SAST |
| **Official** | ⭐ Official (Semgrep) |
| **Source** | https://github.com/semgrep/mcp-semgrep |
| **Transport** | stdio |
| **Install** | `npx -y @semgrep/mcp-server` |

## Tools & Capabilities
- `scan_codebase` — Run local SAST rules across project files to detect OWASP Top 10 vulnerabilities
- `run_custom_rule` — Evaluate custom Semgrep YAML pattern rules against target files
- `explain_vulnerability` — Retrieve CWE classifications, exploit mechanisms, and remediation patches
- `autofix_findings` — Apply verified semantic autofix diffs to resolve security violations

## Client Configuration
```json
{
  "mcpServers": {
    "semgrep": {
      "command": "npx",
      "args": [
        "-y",
        "@semgrep/mcp-server"
      ]
    }
  }
}
```

## Security & Best Practices
- Local AST static analysis. Dry-run verification before applying autofix patches.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
