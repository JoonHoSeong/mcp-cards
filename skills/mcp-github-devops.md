# GitHub MCP Server

## Overview
| Field | Value |
|-------|-------|
| **Name** | GitHub MCP Server |
| **Category** | Development/DevOps |
| **Official** | ⭐ Official (GitHub) |
| **Source** | https://github.com/github/github-mcp-server |
| **Transport** | stdio + remote (https://api.githubcopilot.com/mcp/) |
| **Install** | `docker pull ghcr.io/github/github-mcp-server` OR use hosted remote URL |

## Description
The GitHub MCP Server is GitHub's official implementation that gives AI assistants full access to the GitHub platform — repositories, issues, pull requests, code search, CI/CD workflows, and more. It enables agents to perform complex development operations like creating PRs, searching codebases, triaging issues, and monitoring CI pipelines through natural language commands.

This server supports two deployment modes: a local Docker-based stdio server for private/enterprise use, and a hosted remote endpoint at `https://api.githubcopilot.com/mcp/` for zero-setup integration. The remote mode uses OAuth for authentication, making it ideal for team environments where managing tokens across developers is impractical.

For AI-assisted development workflows, this MCP server is foundational. It transforms conversational requests into precise GitHub API operations — from "create a PR with these changes" to "find all open security alerts in this repo" — with full audit trails and permission controls through GitHub's existing access model.

## Key Tools
| Tool | Description |
|------|-------------|
| search_code | Search for code across repositories using GitHub's code search syntax |
| search_repositories | Find repositories matching specific criteria |
| create_issue | Create a new issue in a repository with labels and assignees |
| create_pull_request | Create a pull request with title, body, and branch references |
| get_pull_request | Retrieve details of a specific pull request |
| list_pull_request_reviews | List all reviews on a pull request |
| create_review_comment | Add a review comment to a pull request |
| get_file_contents | Retrieve file contents from a repository at a specific ref |
| list_commits | List commits on a branch or for a file |
| list_branches | List all branches in a repository |
| get_workflow_run | Get details of a specific GitHub Actions workflow run |
| list_workflow_runs | List recent workflow runs for a repository |

## Configuration
```json
{
  "mcpServers": {
    "github": {
      "command": "docker",
      "args": [
        "run", "-i", "--rm",
        "-e", "GITHUB_PERSONAL_ACCESS_TOKEN",
        "ghcr.io/github/github-mcp-server"
      ],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "<your-token>"
      }
    }
  }
}
```

## Use Cases
1. Automated PR creation — generate pull requests with AI-written descriptions and code changes
2. Code search — find implementations, patterns, or vulnerabilities across repositories
3. Issue triage — categorize, label, and assign incoming issues based on content analysis
4. CI/CD monitoring — check workflow statuses, identify failures, and suggest fixes
5. Changelog generation — summarize commits and PRs into release notes
6. Security alert review — audit Dependabot and code scanning alerts across repos

## NVIDIA Skill Combinations
| Purpose | NVIDIA Skill | Reason |
|---------|-------------|--------|
| Auto-commit generated skill cards | skill-card-generator | Commit and PR generated NVIDIA skill documentation to repos |
| ML framework issue management | mcore-create-issue | Create and track Megatron-Core issues directly from agent conversations |
| Manage model repositories | tao-port-huggingface-model | Handle model porting workflows with GitHub-based version control |

## Prerequisites
- GitHub Personal Access Token (fine-grained) or OAuth app credentials
- Docker runtime (for local stdio deployment)
- Network access to api.github.com or api.githubcopilot.com

## Security Notes
- Use fine-grained Personal Access Tokens with minimal required scopes
- Remote server mode uses OAuth — no token management needed on client side
- Audit all write operations (issue creation, PR creation) through GitHub's audit log
- Consider repository-scoped tokens to limit blast radius
- Never commit tokens to version control — use environment variables or secrets managers

## References
- https://github.com/github/github-mcp-server
- https://docs.github.com/en/rest
- https://github.com/features/copilot
