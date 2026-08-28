# Azure DevOps MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | Azure DevOps MCP Server |
| **Category** | DevOps/CI-CD |
| **Official** | ⭐ Official (Microsoft) |
| **Source** | https://github.com/microsoft/mcp-azure-devops |
| **Transport** | stdio |
| **Install** | `npx -y @microsoft/mcp-azure-devops` |

## Tools & Capabilities
- `list_work_items` — Query Azure Boards work items, bugs, tasks, and sprint iterations
- `create_work_item` — Create and assign new user stories or bug tickets
- `get_pull_requests` — Inspect active Azure Repos PRs, diffs, and review comments
- `trigger_pipeline_build` — Queue automated build and release pipeline execution
- `get_pipeline_status` — Monitor Azure Pipelines run logs, stages, and test summaries

## Client Configuration
```json
{
  "mcpServers": {
    "azure-devops": {
      "command": "npx",
      "args": [
        "-y",
        "@microsoft/mcp-azure-devops"
      ],
      "env": {
        "AZURE_DEVOPS_ORG_URL": "https://dev.azure.com/your-org",
        "AZURE_DEVOPS_PAT": "YOUR_PERSONAL_ACCESS_TOKEN"
      }
    }
  }
}
```

## Security & Best Practices
- Use Scoped Personal Access Tokens (PAT). Restrict release approvals to authorized reviewer roles.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
