# Git MCP Server

## Overview
| Field | Value |
|-------|-------|
| **Name** | Git MCP |
| **Category** | Version Control |
| **Official** | Reference |
| **Source** | https://github.com/modelcontextprotocol/servers/tree/main/src/git |
| **Transport** | stdio |
| **Install** | `npx -y @modelcontextprotocol/server-git` |

## Description
Git MCP server provides local repository analysis capabilities including log inspection, diff generation, status checking, and blame tracking. It enables AI agents to understand code history, review changes, search commit messages, and navigate branch structures without requiring remote API access.

## Key Tools
| Tool | Description |
|------|-------------|
| `git_log` | View commit history with filtering by author, date, and path |
| `git_diff` | Generate diffs between commits, branches, or working tree |
| `git_status` | Check working directory and staging area status |
| `git_show` | Display detailed commit information and content |
| `git_blame` | Show line-by-line authorship of a file |
| `search_commits` | Search commit messages and content by keyword |
| `list_branches` | List local and remote branches |
| `list_tags` | List repository tags with optional filtering |

## Configuration
```json
{
  "mcpServers": {
    "git": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-git", "--repository", "/path/to/repo"]
    }
  }
}
```

## Use Cases
1. Analyze code history to understand architectural decisions  2. Review changes before committing or creating merge requests  3. Track down regressions using blame and commit search

## NVIDIA Skill Combinations
| Purpose | NVIDIA Skill | Reason |
|---------|-------------|--------|
| Analyze NeMo repos locally | ALL | Provides local git context for any NVIDIA skill working with source code |
| Review Megatron-Core changes | mcore-split-pr | Understand commit history before splitting large PRs |
| Track DOCA SDK changes | doca-programming-guide | Correlate code changes with documentation updates |

## Prerequisites
- Git installed on the system
- Valid Git repository path accessible to the server

## Security Notes
- Only exposes read-only git operations (no push, commit, or reset)
- Repository path must be explicitly configured; no arbitrary filesystem access
- Sensitive files in repo history (secrets, keys) may be exposed through git_show

## References
- https://github.com/modelcontextprotocol/servers/tree/main/src/git
- https://git-scm.com/docs
