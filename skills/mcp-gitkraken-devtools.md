# GitKraken MCP Server

## Overview
| Field | Value |
|-------|-------|
| **Name** | GitKraken MCP |
| **Category** | DevTools |
| **Official** | Official |
| **Source** | https://github.com/gitkraken/gk-cli |
| **Transport** | stdio |
| **Install** | `gk mcp serve` |

## Description
GitKraken MCP server provides a unified interface across multiple Git hosting platforms (GitHub, GitLab, Bitbucket) and project management tools (Jira, Trello). Through the GitKraken CLI, AI agents can manage repositories, pull requests, issues, and code search across all connected platforms from a single MCP server instance.

## Key Tools
| Tool | Description |
|------|-------------|
| `list_repos` | List repositories across all connected platforms |
| `get_repo_info` | Get detailed repository information and statistics |
| `list_pull_requests` | List PRs across GitHub, GitLab, and Bitbucket |
| `create_pr` | Create a pull request on any connected platform |
| `list_issues` | List issues from GitHub, GitLab, Jira, or Trello |
| `search_code` | Search code across all connected repositories |
| `get_git_graph` | Visualize branch topology and merge history |

## Configuration
```json
{
  "mcpServers": {
    "gitkraken": {
      "command": "gk",
      "args": ["mcp", "serve"]
    }
  }
}
```

## Use Cases
1. Unified PR management across GitHub, GitLab, and Bitbucket  2. Cross-platform issue tracking and synchronization  3. Code search across all team repositories regardless of hosting platform

## NVIDIA Skill Combinations
| Purpose | NVIDIA Skill | Reason |
|---------|-------------|--------|
| Cross-platform issues | mcore-create-issue | Create and track Megatron-Core issues across GitHub and internal GitLab |
| Multi-repo PR management | mcore-split-pr | Coordinate split PRs across multiple repositories and platforms |
| Code search for NVIDIA repos | ALL | Search NVIDIA open-source repositories across GitHub for reference code |

## Prerequisites
- GitKraken CLI (`gk`) installed and authenticated
- At least one Git hosting platform connected (GitHub, GitLab, or Bitbucket)

## Security Notes
- gk CLI stores credentials locally; ensure secure file permissions on config
- Cross-platform access means a single compromise exposes all connected services
- Code search results may include sensitive code; filter by repository visibility

## References
- https://github.com/gitkraken/gk-cli
- https://www.gitkraken.com/cli
