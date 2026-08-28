# GitLab MCP Server

## Overview
| Field | Value |
|-------|-------|
| **Name** | GitLab MCP |
| **Category** | DevOps |
| **Official** | Reference |
| **Source** | https://github.com/modelcontextprotocol/servers/tree/main/src/gitlab |
| **Transport** | stdio |
| **Install** | `npx -y @modelcontextprotocol/server-gitlab` |

## Description
GitLab MCP server provides comprehensive access to the GitLab API for managing repositories, merge requests, issues, pipelines, and branches. It enables AI agents to automate DevOps workflows including code review, issue triage, CI/CD monitoring, and project management directly through the Model Context Protocol.

## Key Tools
| Tool | Description |
|------|-------------|
| `list_projects` | List accessible GitLab projects with filtering options |
| `get_project` | Get detailed project information including settings and statistics |
| `list_merge_requests` | List merge requests with state and label filters |
| `create_merge_request` | Create a new merge request with title, description, and reviewers |
| `list_issues` | List project issues with milestone and label filtering |
| `create_issue` | Create a new issue with labels, assignees, and milestones |
| `get_pipeline_status` | Get the status of CI/CD pipelines for a project |
| `list_branches` | List repository branches with search and protection status |

## Configuration
```json
{
  "mcpServers": {
    "gitlab": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-gitlab"],
      "env": {
        "GITLAB_PERSONAL_ACCESS_TOKEN": "<your-token>",
        "GITLAB_API_URL": "https://gitlab.com/api/v4"
      }
    }
  }
}
```

## Use Cases
1. Automate merge request creation and code review workflows  2. Monitor CI/CD pipeline status and triage failures  3. Bulk issue management and cross-project tracking

## NVIDIA Skill Combinations
| Purpose | NVIDIA Skill | Reason |
|---------|-------------|--------|
| Track Megatron-Core issues | mcore-create-issue | Sync GitLab issues with NVIDIA internal tracking for NeMo/Megatron development |
| Monitor training pipelines | mcore-split-pr | Coordinate large PRs across GitLab CI pipeline stages |

## Prerequisites
- `GITLAB_PERSONAL_ACCESS_TOKEN` with api scope
- GitLab instance URL (defaults to gitlab.com)

## Security Notes
- Store tokens in environment variables, never in configuration files
- Use minimal token scopes (read_api for read-only workflows)
- Restrict API URL to your specific GitLab instance

## References
- https://github.com/modelcontextprotocol/servers/tree/main/src/gitlab
- https://docs.gitlab.com/ee/api/
