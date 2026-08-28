# Neon Serverless Postgres MCP Server

## Overview
| Field | Value |
|-------|-------|
| **Name** | mcp-neon-serverless-pg |
| **Category** | Database |
| **Official** | Official |
| **Source** | https://github.com/neondatabase/mcp-server-neon |
| **Transport** | stdio |
| **Install** | `npx -y @neondatabase/mcp-server-neon` |

## Description
The Neon MCP server provides programmatic access to Neon's serverless PostgreSQL platform, enabling database creation, branch management, and SQL execution. Neon's branching model allows instant database copies for development, testing, and preview environments, making it ideal for AI agent workflows that need isolated, disposable database instances with zero cold-start latency.

## Key Tools
| Tool | Description |
|------|-------------|
| `create_database` | Create a new Neon PostgreSQL database |
| `list_databases` | List all databases in the Neon project |
| `execute_sql` | Execute SQL statements against a Neon database |
| `create_branch` | Create a database branch for isolated development |
| `list_branches` | List all branches in a project |
| `get_connection_string` | Retrieve the connection string for a database branch |

## Configuration
```json
{
  "mcpServers": {
    "neon": {
      "command": "npx",
      "args": ["-y", "@neondatabase/mcp-server-neon"],
      "env": {
        "NEON_API_KEY": "your-neon-api-key"
      }
    }
  }
}
```

## Use Cases
1. Spinning up isolated database branches for agent testing and experimentation
2. Managing serverless PostgreSQL backends for AI-powered applications
3. Creating ephemeral databases for CI/CD pipeline integration tests

## NVIDIA Skill Combinations
| Purpose | NVIDIA Skill | Reason |
|---------|-------------|--------|
| Serverless DB for agents | aiq-deploy | Provision on-demand PostgreSQL instances for deployed AIQ agent backends |

## Prerequisites
- Neon account with an active project
- `NEON_API_KEY` from the Neon console

## Security Notes
- API keys grant full project access; rotate regularly and use separate keys per environment
- Database branches inherit permissions from the parent; scope access appropriately
- Connection strings contain credentials; never log or expose them

## References
- https://github.com/neondatabase/mcp-server-neon
- https://neon.tech/docs
