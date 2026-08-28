# MySQL MCP Server

## Overview
| Field | Value |
|-------|-------|
| **Name** | mcp-mysql-db |
| **Category** | Database |
| **Official** | Community |
| **Source** | https://github.com/designcomputer/mysql_mcp_server |
| **Transport** | stdio |
| **Install** | `uvx mysql_mcp_server` |

## Description
The MySQL MCP server enables AI agents to connect to and query MySQL databases, supporting SQL execution, schema inspection, table listing, and database enumeration. It provides a straightforward interface for interacting with MySQL instances, making it suitable for application development, database administration, and data exploration tasks against existing MySQL deployments.

## Key Tools
| Tool | Description |
|------|-------------|
| `execute_query` | Execute SQL queries (SELECT, INSERT, UPDATE, DELETE) |
| `list_databases` | List all databases on the MySQL server |
| `list_tables` | List all tables in a specified database |
| `describe_table` | Get column definitions and key information |
| `get_schema` | Retrieve full schema DDL for tables |

## Configuration
```json
{
  "mcpServers": {
    "mysql": {
      "command": "uvx",
      "args": ["mysql_mcp_server"],
      "env": {
        "MYSQL_HOST": "localhost",
        "MYSQL_PORT": "3306",
        "MYSQL_USER": "your-username",
        "MYSQL_PASSWORD": "your-password",
        "MYSQL_DATABASE": "your-database"
      }
    }
  }
}
```

## Use Cases
1. Querying and managing existing MySQL application databases
2. Schema exploration and documentation of legacy MySQL deployments
3. Running data migrations and transformation queries

## NVIDIA Skill Combinations
| Purpose | NVIDIA Skill | Reason |
|---------|-------------|--------|
| MySQL model config store | nemotron-customize | Store and retrieve model customization configurations and training metadata in MySQL |

## Prerequisites
- MySQL server instance (5.7+ or 8.x)
- `MYSQL_HOST`, `MYSQL_USER`, and `MYSQL_PASSWORD` credentials
- Network access to the MySQL port (default 3306)

## Security Notes
- Create dedicated database users with least-privilege access for agent connections
- Use SSL/TLS connections for remote MySQL instances
- Avoid granting DROP or ALTER privileges unless explicitly required

## References
- https://github.com/designcomputer/mysql_mcp_server
- https://dev.mysql.com/doc/
