# PlanetScale MCP Server

## Overview
| Field | Value |
|-------|-------|
| **Name** | mcp-planetscale-mysql |
| **Category** | Database |
| **Official** | Official (hosted remote, OAuth) |
| **Source** | https://planetscale.com/docs/connect/mcp |
| **Transport** | stdio |
| **Install** | OAuth-based hosted remote MCP (no local install) |

## Description
The PlanetScale MCP server provides AI agents with managed MySQL database access through PlanetScale's serverless platform. It supports database and branch management, schema inspection, query execution, performance insights, and deploy requests for non-blocking schema changes. The OAuth-based authentication model eliminates credential management while providing secure, scoped access to PlanetScale resources.

## Key Tools
| Tool | Description |
|------|-------------|
| `list_databases` | List all databases in the PlanetScale organization |
| `list_branches` | List database branches for a given database |
| `get_schema` | Retrieve the schema for a specific branch |
| `execute_query` | Execute SQL queries against a database branch |
| `create_branch` | Create a new database branch for development |
| `get_insights` | Get query performance insights and recommendations |
| `deploy_request` | Create a deploy request for schema changes |

## Configuration
```json
{
  "mcpServers": {
    "planetscale": {
      "command": "npx",
      "args": ["-y", "@planetscale/mcp"],
      "env": {}
    }
  }
}
```

## Use Cases
1. Managing MySQL schema changes with non-blocking deploy requests
2. Branching databases for isolated feature development and testing
3. Monitoring query performance and applying optimization insights

## NVIDIA Skill Combinations
| Purpose | NVIDIA Skill | Reason |
|---------|-------------|--------|
| Managed MySQL for model configs | nemotron-customize | Store model fine-tuning configurations and experiment metadata in managed MySQL |

## Prerequisites
- PlanetScale account with at least one database
- OAuth authentication (configured through PlanetScale dashboard)
- No local API keys required; auth handled via OAuth flow

## Security Notes
- OAuth scopes should be limited to required databases and operations
- Deploy requests provide review gates before schema changes reach production
- Branch-level access control prevents accidental production modifications

## References
- https://planetscale.com/docs/connect/mcp
- https://planetscale.com/docs
