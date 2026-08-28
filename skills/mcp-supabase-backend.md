# Supabase MCP Server

## Overview
| Field | Value |
|-------|-------|
| **Name** | Supabase MCP Server |
| **Category** | Backend/BaaS |
| **Official** | ⭐ Official (Supabase) |
| **Source** | https://github.com/supabase-community/mcp-supabase |
| **Transport** | stdio |
| **Install** | `npx -y @supabase/mcp` |

## Description
Official Supabase MCP server for full-stack backend management. Enables AI agents to manage databases, execute SQL, handle storage operations, deploy edge functions, and monitor projects. Provides comprehensive access to the Supabase platform including Postgres, Auth, Storage, and Edge Functions.

## Key Tools
| Tool | Description |
|------|-------------|
| `list_tables` | List all tables in the database |
| `execute_sql` | Execute arbitrary SQL queries against the database |
| `get_table_schema` | Get the schema definition of a specific table |
| `list_functions` | List all database functions/stored procedures |
| `invoke_function` | Invoke a database function with parameters |
| `list_storage_buckets` | List all storage buckets in the project |
| `upload_file` | Upload a file to a storage bucket |
| `list_edge_functions` | List all deployed edge functions |
| `deploy_edge_function` | Deploy or update an edge function |
| `get_project_info` | Get project configuration and status |

## Configuration
```json
{
  "mcpServers": {
    "supabase": {
      "command": "npx",
      "args": ["-y", "@supabase/mcp"],
      "env": {
        "SUPABASE_ACCESS_TOKEN": "your-access-token",
        "SUPABASE_PROJECT_REF": "your-project-ref"
      }
    }
  }
}
```

## Use Cases
1. Full-stack backend management — tables, functions, and storage from IDE
2. Database migrations via chat — generate and execute schema changes
3. Storage operations — upload, organize, and manage files
4. Edge function deployment — write and deploy serverless functions
5. Project monitoring — check status, quotas, and configuration

## NVIDIA Skill Combinations
| Purpose | NVIDIA Skill | Reason |
|---------|-------------|--------|
| pgvector for RAG | `rag-blueprint` | Use Supabase pgvector extension as the vector store backend for RAG pipelines |
| Backend for AI-Q agents | `aiq-deploy` | Deploy AI-Q agent backends with Supabase as the data layer |
| Store synthetic data | `data-designer` | Persist generated synthetic datasets in structured Postgres tables |

## Prerequisites
- Active Supabase project (free tier available)
- `SUPABASE_ACCESS_TOKEN` — personal access token from dashboard
- Project reference ID from Supabase dashboard URL
- Node.js 18+ for npx installation

## Security Notes
- Use `service_role` key only in server-side contexts, never expose to clients
- Enable Row-Level Security (RLS) on all tables with sensitive data
- Access token has full project access — treat as a secret
- Use database roles to restrict SQL execution scope
- Audit all `execute_sql` calls in production environments

## References
- [GitHub Repository](https://github.com/supabase-community/mcp-supabase)
- [Supabase Documentation](https://supabase.com/docs)
- [Supabase pgvector Guide](https://supabase.com/docs/guides/ai)
