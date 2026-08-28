# Linear MCP Server

## Overview
| Field | Value |
|-------|-------|
| **Name** | Linear MCP |
| **Category** | Project Management |
| **Official** | Official |
| **Source** | https://github.com/jerhadf/linear-mcp-server |
| **Transport** | stdio |
| **Install** | `npx -y linear-mcp-server` |

## Description
Linear MCP server enables AI agents to manage issues, projects, and development cycles in Linear. It provides comprehensive project management capabilities including issue creation, sprint tracking, team workload visibility, and search, allowing AI-powered development workflows to stay synchronized with team planning and progress tracking.

## Key Tools
| Tool | Description |
|------|-------------|
| `list_issues` | List issues with filtering by state, assignee, and label |
| `create_issue` | Create a new issue with title, description, and metadata |
| `update_issue` | Update issue fields including state, priority, and assignee |
| `list_projects` | List projects with progress and status information |
| `list_cycles` | List development cycles (sprints) with dates |
| `get_cycle_progress` | Get completion metrics for a specific cycle |
| `list_teams` | List teams and their settings |
| `search_issues` | Full-text search across all issues |

## Configuration
```json
{
  "mcpServers": {
    "linear": {
      "command": "npx",
      "args": ["-y", "linear-mcp-server"],
      "env": {
        "LINEAR_API_KEY": "<your-api-key>"
      }
    }
  }
}
```

## Use Cases
1. Automatically create issues from code review findings  2. Track sprint progress and identify blocked work items  3. Link pull requests to Linear issues for traceability

## NVIDIA Skill Combinations
| Purpose | NVIDIA Skill | Reason |
|---------|-------------|--------|
| Link PRs to issues | mcore-split-pr | Automatically link split Megatron-Core PRs to tracking issues in Linear |
| Track model training tasks | nemotron-customize | Create Linear issues for model customization tasks and track progress |
| Issue from CI failures | mcore-create-issue | Auto-create Linear issues when NVIDIA CI pipelines detect failures |

## Prerequisites
- `LINEAR_API_KEY` from Linear Settings → API
- Linear workspace with teams and projects configured

## Security Notes
- API key provides access to all workspace data; use personal keys with caution
- Issue content may contain sensitive technical details; restrict workspace access
- Webhook integrations should verify Linear's signing secret

## References
- https://github.com/jerhadf/linear-mcp-server
- https://developers.linear.app/docs
