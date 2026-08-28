# E2B MCP Server

## Overview
| Field | Value |
|-------|-------|
| **Name** | E2B MCP |
| **Category** | Sandbox/Code Execution |
| **Official** | Yes |
| **Source** | https://github.com/e2b-dev/mcp-server |
| **Transport** | stdio |
| **Install** | `npx @e2b/mcp-server` |

## Description
The E2B MCP server provides AI agents with secure sandboxed environments for code execution. It enables running arbitrary code, shell commands, and file operations in isolated containers, making it ideal for safe experimentation, data processing, and code generation validation.

## Key Tools
| Tool | Description |
|------|-------------|
| `create_sandbox` | Create a new isolated sandbox environment |
| `run_code` | Execute code in the sandbox (Python, JS, etc.) |
| `run_command` | Run shell commands in sandbox |
| `upload_file` | Upload a file to the sandbox |
| `download_file` | Download a file from the sandbox |
| `list_files` | List files in sandbox filesystem |
| `kill_sandbox` | Terminate a sandbox instance |

## Configuration
```json
{
  "mcpServers": {
    "e2b": {
      "command": "npx",
      "args": ["@e2b/mcp-server"],
      "env": {
        "E2B_API_KEY": "${E2B_API_KEY}"
      }
    }
  }
}
```

## Use Cases
1. Safe execution of AI-generated code in isolated environments
2. Data processing and transformation pipelines
3. Automated testing and validation of code artifacts

## NVIDIA Skill Combinations
| Purpose | NVIDIA Skill | Reason |
|---------|-------------|--------|
| Safe synthetic data gen | data-designer | Execute data generation scripts in sandboxed environments for safety |
| Safe RL code execution | nemo-rl-auto-research | Run reinforcement learning experiments in isolated containers |

## Prerequisites
- `E2B_API_KEY` — E2B API key for sandbox provisioning

## Security Notes
- Sandboxes are isolated but API key usage is metered; monitor for abuse
- Uploaded files are ephemeral; sandbox destruction removes all data

## References
- https://github.com/e2b-dev/mcp-server
- https://e2b.dev/docs
