# PostgreSQL MCP Server

## Overview
| Field | Value |
|-------|-------|
| **Name** | PostgreSQL MCP Server |
| **Category** | Database |
| **Official** | ⭐ Reference Implementation |
| **Source** | https://github.com/modelcontextprotocol/servers/tree/main/src/postgres |
| **Transport** | stdio |
| **Install** | `npx -y @modelcontextprotocol/server-postgres` |

## Description
The PostgreSQL MCP Server is the official reference implementation for database access via the Model Context Protocol. It provides AI assistants with the ability to query PostgreSQL databases, explore schemas, and retrieve data — all through natural language interactions that get translated into safe, read-only SQL queries.

This server is essential for data exploration workflows where users need to quickly understand database structures, run ad-hoc analytics queries, or validate data without writing SQL manually. It exposes table structures, relationships, and data through a controlled interface that prevents destructive operations by default.

As part of the official MCP servers collection maintained by Anthropic, this implementation serves as both a production-ready tool and a reference for building custom database MCP servers. It handles connection pooling, query timeouts, and result formatting automatically.

## Key Tools
| Tool | Description |
|------|-------------|
| query | Execute read-only SQL queries against the connected PostgreSQL database |
| list_tables | List all tables in the database with their schemas |
| describe_table | Get detailed column information, types, and constraints for a table |
| get_schema | Retrieve the complete database schema including relationships |

## Configuration
```json
{
  "mcpServers": {
    "postgres": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-postgres",
        "postgresql://user:password@localhost:5432/dbname"
      ]
    }
  }
}
```

## Use Cases
1. Natural language database queries — ask questions about data without writing SQL
2. Schema exploration — understand table structures, relationships, and column types interactively
3. Analytics dashboards via chat — generate reports and summaries from database content
4. Data validation — verify data integrity and check for anomalies through conversational queries

## NVIDIA Skill Combinations
| Purpose | NVIDIA Skill | Reason |
|---------|-------------|--------|
| Store and query embeddings | rag-blueprint | PostgreSQL with pgvector stores RAG embeddings for retrieval pipelines |
| Store evaluation results | nemo-evaluator-plugin | Persist NeMo model evaluation metrics and results in structured tables |
| Video analytics data | vss-query-analytics | Query structured analytics data from video surveillance pipelines |

## Prerequisites
- PostgreSQL database with network accessibility
- Valid PostgreSQL connection string (user, password, host, port, database)
- Node.js runtime for npx execution

## Security Notes
- Default mode is read-only — SELECT queries only, no INSERT/UPDATE/DELETE
- Never expose database credentials in version-controlled configuration files
- Use environment variables or secrets managers for connection strings
- Consider creating a dedicated read-only database user for MCP access
- Limit network access to the database from the MCP server host only

## References
- https://github.com/modelcontextprotocol/servers/tree/main/src/postgres
- https://modelcontextprotocol.io/
- https://www.postgresql.org/docs/
