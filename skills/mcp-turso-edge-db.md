# Turso Edge Database MCP Server

## Overview
| Field | Value |
|-------|-------|
| **Name** | mcp-turso-edge-db |
| **Category** | Database |
| **Official** | Community |
| **Source** | https://github.com/spences10/mcp-turso-cloud |
| **Transport** | stdio |
| **Install** | `npx -y mcp-turso-cloud` |

## Description
The Turso MCP server provides AI agents with access to Turso's edge-native SQLite-compatible database platform (built on libSQL). It supports database creation, SQL execution, group management, and token generation. Turso replicates databases to edge locations worldwide, delivering ultra-low-latency reads ideal for edge computing, IoT, and embedded AI applications that need local-speed data access.

## Key Tools
| Tool | Description |
|------|-------------|
| `list_databases` | List all databases in the organization |
| `create_database` | Create a new edge-replicated database |
| `execute_query` | Execute SQL queries against a Turso database |
| `list_groups` | List database groups and their regions |
| `get_db_stats` | Get storage and usage statistics for a database |
| `create_token` | Generate scoped authentication tokens for databases |

## Configuration
```json
{
  "mcpServers": {
    "turso": {
      "command": "npx",
      "args": ["-y", "mcp-turso-cloud"],
      "env": {
        "TURSO_API_TOKEN": "your-turso-api-token",
        "TURSO_ORG": "your-organization-slug"
      }
    }
  }
}
```

## Use Cases
1. Deploying edge-replicated databases for low-latency IoT applications
2. Managing SQLite-compatible databases across global edge locations
3. Generating scoped tokens for secure multi-tenant database access

## NVIDIA Skill Combinations
| Purpose | NVIDIA Skill | Reason |
|---------|-------------|--------|
| Edge database for Jetson | jetson-llm-serve | Provide local-latency data access for LLM serving on Jetson edge devices |

## Prerequisites
- Turso account with organization created
- `TURSO_API_TOKEN` from the Turso CLI or dashboard
- `TURSO_ORG` organization slug

## Security Notes
- API tokens grant organization-wide access; use database-scoped tokens for applications
- Generated tokens should have expiration times set for production use
- Edge replicas inherit the primary's access controls

## References
- https://github.com/spences10/mcp-turso-cloud
- https://docs.turso.tech/
