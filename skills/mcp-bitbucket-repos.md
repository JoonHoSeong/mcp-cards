# Bitbucket Repositories & CI/CD MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | Bitbucket Repositories & CI/CD MCP Server |
| **Category** | DevOps/Git |
| **Official** | ⭐ Official (Atlassian Bitbucket) |
| **Source** | https://github.com/atlassian/mcp-bitbucket |
| **Transport** | stdio |
| **Install** | `npx -y @atlassian/mcp-bitbucket` |

## Tools & Capabilities
- `list_repositories` — Query workspace repositories, projects, and main branch settings
- `get_pull_requests` — Inspect open PRs, diff files, approval status, and reviewers
- `create_pull_request` — Create new pull request with title, description, and source/target branches
- `trigger_pipeline` — Dispatch Bitbucket Pipelines build execution for specific commit
- `get_pipeline_logs` — Inspect step build logs and failure diagnostics

## Client Configuration
```json
{
  "mcpServers": {
    "bitbucket": {
      "command": "npx",
      "args": [
        "-y",
        "@atlassian/mcp-bitbucket"
      ],
      "env": {
        "BITBUCKET_USERNAME": "your-username",
        "BITBUCKET_APP_PASSWORD": "YOUR_APP_PASSWORD",
        "BITBUCKET_WORKSPACE": "your-workspace"
      }
    }
  }
}
```

## Security & Best Practices
- Use Scoped Bitbucket App Passwords with repository and pipeline permissions.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
