# Prisma MCP Server

## Overview
| Field | Value |
|-------|-------|
| **Name** | mcp-prisma-orm |
| **Category** | Database |
| **Official** | Official |
| **Source** | https://github.com/prisma/mcp |
| **Transport** | stdio |
| **Install** | `npx -y @prisma/mcp` |

## Description
The Prisma MCP server enables AI agents to introspect, migrate, and query databases through Prisma's type-safe ORM layer. It supports schema inspection, migration generation and execution, database querying, and schema validation across PostgreSQL, MySQL, SQLite, and other Prisma-supported databases. This makes it ideal for AI-assisted database development workflows with full schema awareness.

## Key Tools
| Tool | Description |
|------|-------------|
| `introspect_schema` | Pull the current database schema into Prisma format |
| `generate_migration` | Generate a migration from schema changes |
| `run_migration` | Apply pending migrations to the database |
| `query_database` | Execute queries through Prisma's query engine |
| `format_schema` | Format and normalize the Prisma schema file |
| `validate_schema` | Validate the schema for errors and warnings |

## Configuration
```json
{
  "mcpServers": {
    "prisma": {
      "command": "npx",
      "args": ["-y", "@prisma/mcp"],
      "env": {
        "DATABASE_URL": "postgresql://user:pass@localhost:5432/mydb"
      }
    }
  }
}
```

## Use Cases
1. AI-assisted database schema design and migration workflows
2. Introspecting existing databases to generate type-safe access layers
3. Validating and formatting schema files in CI/CD pipelines

## NVIDIA Skill Combinations
| Purpose | NVIDIA Skill | Reason |
|---------|-------------|--------|
| Schema-driven synthetic data | data-designer | Use introspected Prisma schemas to generate realistic synthetic data matching production models |

## Prerequisites
- Prisma project with `schema.prisma` file
- `DATABASE_URL` pointing to a supported database (PostgreSQL, MySQL, SQLite, etc.)
- Node.js runtime

## Security Notes
- Database URLs contain credentials; store in environment variables only
- Migration operations can be destructive; use with caution on production databases
- Validate migrations in staging before applying to production

## References
- https://github.com/prisma/mcp
- https://www.prisma.io/docs
