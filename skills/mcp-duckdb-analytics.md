# DuckDB MCP Server

## Overview
| Field | Value |
|-------|-------|
| **Name** | mcp-duckdb-analytics |
| **Category** | Database |
| **Official** | Community |
| **Source** | https://github.com/ktanaka101/mcp-server-duckdb |
| **Transport** | stdio |
| **Install** | `uvx mcp-server-duckdb` |

## Description
The DuckDB MCP server provides AI agents with an embedded analytical database engine optimized for OLAP workloads. It supports SQL execution, direct import from CSV and Parquet files, data export, and schema inspection. DuckDB runs in-process without a separate server, making it ideal for local data analysis, ETL pipelines, and rapid exploration of file-based datasets without infrastructure overhead.

## Key Tools
| Tool | Description |
|------|-------------|
| `execute_query` | Execute analytical SQL queries on DuckDB |
| `list_tables` | List all tables in the current database |
| `describe_table` | Get column types and table metadata |
| `import_csv` | Import CSV files directly into DuckDB tables |
| `import_parquet` | Import Parquet files for columnar analytics |
| `export_data` | Export query results to various file formats |

## Configuration
```json
{
  "mcpServers": {
    "duckdb": {
      "command": "uvx",
      "args": ["mcp-server-duckdb", "--db-path", "/path/to/analytics.duckdb"],
      "env": {}
    }
  }
}
```

## Use Cases
1. Local OLAP analytics on CSV and Parquet datasets without external infrastructure
2. ETL pipeline prototyping with direct file imports and SQL transformations
3. Rapid data exploration and profiling for machine learning feature engineering

## NVIDIA Skill Combinations
| Purpose | NVIDIA Skill | Reason |
|---------|-------------|--------|
| Local analytics engine | data-designer | Analyze and validate synthetic datasets locally using DuckDB's fast columnar queries |

## Prerequisites
- DuckDB file path (or use in-memory mode with no path)
- Python environment with `uvx` available
- Source data files (CSV, Parquet) for import operations

## Security Notes
- DuckDB has full filesystem access for imports/exports; restrict the working directory
- In-memory mode leaves no persistent data; use for sensitive exploratory analysis
- Database files are unencrypted by default; protect with filesystem permissions

## References
- https://github.com/ktanaka101/mcp-server-duckdb
- https://duckdb.org/docs/
