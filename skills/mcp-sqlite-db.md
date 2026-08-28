# SQLite MCP Server

## Overview
| Field | Value |
|-------|-------|
| **Name** | mcp-sqlite-db |
| **Category** | Database |
| **Official** | Reference |
| **Source** | https://github.com/modelcontextprotocol/servers/tree/main/src/sqlite |
| **Transport** | stdio |
| **Install** | `uvx mcp-server-sqlite` |

## Description
The SQLite MCP server provides a lightweight, file-based relational database interface for AI agents. It enables schema inspection, table creation, and SQL query execution against local SQLite databases, making it ideal for prototyping, local data analysis, and embedded application storage without requiring external database infrastructure.

## Key Tools
| Tool | Description |
|------|-------------|
| `query` | Execute read-only SQL queries against the database |
| `list_tables` | List all tables in the connected SQLite database |
| `describe_table` | Get schema details including columns and types for a table |
| `create_table` | Create a new table with specified columns and constraints |
| `write_query` | Execute INSERT, UPDATE, or DELETE statements |

## Configuration
```json
{
  "mcpServers": {
    "sqlite": {
      "command": "uvx",
      "args": ["mcp-server-sqlite", "--db-path", "/path/to/database.db"],
      "env": {}
    }
  }
}
```

## Use Cases
1. Rapid prototyping of data models and schema designs locally
2. Managing embedded application databases for lightweight services
3. Querying and transforming local datasets for analysis pipelines

## NVIDIA Skill Combinations
| Purpose | NVIDIA Skill | Reason |
|---------|-------------|--------|
| Local synthetic data storage | data-designer | Store and query generated synthetic datasets locally for testing and validation |

## Prerequisites
- SQLite file path (existing or new database location)
- Python environment with `uvx` available

## Security Notes
- Database file permissions should be restricted to prevent unauthorized access
- Write operations are unrestricted; use read-only mode for sensitive data exploration

## References
- https://github.com/modelcontextprotocol/servers/tree/main/src/sqlite
