# GitHub Actions CI/CD MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | GitHub Actions CI/CD MCP Server |
| **Category** | DevOps/CI-CD |
| **Official** | ⭐ Official / Verified |
| **Source** | https://github.com/modelcontextprotocol/servers/tree/main/src/github-actions |
| **Transport** | stdio |
| **Install** | `npx -y @modelcontextprotocol/server-github-actions` |

## Tools & Capabilities
- `dispatch_workflow` — Trigger workflow dispatch events with custom input parameters
- `get_workflow_run_logs` — Stream real-time step failure traces and test logs
- `list_workflow_runs` — Query recent runs, commit hashes, branches, and status (success/failure)
- `download_artifact` — Retrieve build artifacts, test summaries, and binaries

## Client Configuration
```json
{
  "mcpServers": {
    "github-actions": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-github-actions"
      ],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "ghp_..."
      }
    }
  }
}
```

## Security & Best Practices
- Fine-grained Personal Access Token with `actions:read` and `actions:write` scopes.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
