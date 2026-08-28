# Vercel MCP Server

## Overview
| Field | Value |
|-------|-------|
| **Name** | Vercel MCP |
| **Category** | Cloud/Deployment |
| **Official** | Yes (Vercel) |
| **Source** | https://github.com/mcp/com.vercel/vercel-mcp |
| **Transport** | stdio |
| **Install** | `npx @vercel/mcp-server` |

## Description
The Vercel MCP Server provides AI agents with deployment and project management capabilities on the Vercel platform. It enables automated deployments, environment variable management, domain configuration, and deployment log inspection — ideal for shipping AI-powered frontend applications and serverless APIs with zero-config infrastructure.

## Key Tools
| Tool | Description |
|------|-------------|
| `list_projects` | List all projects in the Vercel account |
| `get_project` | Get detailed information about a specific project |
| `list_deployments` | List recent deployments for a project |
| `create_deployment` | Trigger a new deployment |
| `get_deployment_logs` | Retrieve build and runtime logs |
| `list_domains` | List custom domains for a project |
| `list_env_vars` | List environment variables for a project |
| `set_env_var` | Create or update an environment variable |

## Configuration
```json
{
  "mcpServers": {
    "vercel": {
      "command": "npx",
      "args": ["@vercel/mcp-server"],
      "env": {
        "VERCEL_TOKEN": "your-vercel-token",
        "VERCEL_TEAM_ID": "your-team-id"
      }
    }
  }
}
```

## Use Cases
1. Deploy AI-powered frontend applications with automatic preview URLs
2. Manage environment variables for NVIDIA API keys across environments
3. Monitor deployment health and rollback failed releases automatically

## NVIDIA Skill Combinations
| Purpose | NVIDIA Skill | Reason |
|---------|-------------|--------|
| AI frontend deploy | aiq-deploy | Deploy AI agent interfaces and chat UIs to Vercel edge |
| Environment config | aiq-deploy | Manage NVIDIA API endpoints as Vercel env vars |
| Preview deployments | aiq-deploy | Test AI features in isolated preview environments |

## Prerequisites
- Vercel account (Pro or Enterprise for team features)
- `VERCEL_TOKEN` personal access token
- Team ID for organization-scoped operations

## Security Notes
- Use scoped tokens that limit access to specific projects
- Store sensitive env vars as encrypted secrets in Vercel
- Enable deployment protection for production environments
- Audit token usage through Vercel's activity log

## References
- https://github.com/mcp/com.vercel/vercel-mcp
- https://vercel.com/docs/rest-api
- https://vercel.com/docs/deployments
