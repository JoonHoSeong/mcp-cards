# Snowflake MCP Server

## Overview
| Field | Value |
|-------|-------|
| **Name** | mcp-snowflake-warehouse |
| **Category** | Database |
| **Official** | Community |
| **Source** | https://github.com/isaacwasserman/mcp-snowflake-server |
| **Transport** | stdio |
| **Install** | `uvx mcp-snowflake-server` |

## Description
The Snowflake MCP server enables AI agents to interact with Snowflake's cloud data warehouse, supporting SQL query execution, schema exploration, query history inspection, and warehouse management. It provides access to Snowflake's separation of storage and compute, enabling cost-effective analytics on large-scale structured and semi-structured data for enterprise data workflows.

## Key Tools
| Tool | Description |
|------|-------------|
| `execute_query` | Run SQL queries against Snowflake warehouses |
| `list_schemas` | List all schemas within a database |
| `list_tables` | List tables and views in a schema |
| `describe_table` | Get column definitions, types, and constraints |
| `get_query_history` | Retrieve recent query execution history and performance |
| `list_warehouses` | List available compute warehouses and their status |

## Configuration
```json
{
  "mcpServers": {
    "snowflake": {
      "command": "uvx",
      "args": ["mcp-snowflake-server"],
      "env": {
        "SNOWFLAKE_ACCOUNT": "your-account.region",
        "SNOWFLAKE_USER": "your-username",
        "SNOWFLAKE_PASSWORD": "your-password",
        "SNOWFLAKE_WAREHOUSE": "COMPUTE_WH",
        "SNOWFLAKE_DATABASE": "MY_DB"
      }
    }
  }
}
```

## Use Cases
1. Querying enterprise data warehouses for business intelligence and reporting
2. Exploring and documenting existing Snowflake schemas and data models
3. Analyzing query performance and optimizing warehouse utilization

## NVIDIA Skill Combinations
| Purpose | NVIDIA Skill | Reason |
|---------|-------------|--------|
| Financial data warehouse | portfolio-optimization | Query financial market data and portfolio holdings stored in Snowflake for optimization models |

## Prerequisites
- Snowflake account with warehouse access
- `SNOWFLAKE_ACCOUNT` identifier (account.region format)
- `SNOWFLAKE_USER` and `SNOWFLAKE_PASSWORD` credentials
- Active compute warehouse for query execution

## Security Notes
- Use key-pair authentication over password-based auth for production deployments
- Configure warehouse auto-suspend to prevent runaway compute costs
- Apply row-level security policies and column masking for sensitive data access

## References
- https://github.com/isaacwasserman/mcp-snowflake-server
- https://docs.snowflake.com/
