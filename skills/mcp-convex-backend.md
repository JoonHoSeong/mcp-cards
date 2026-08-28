# Convex MCP Server

## Overview
| Field | Value |
|-------|-------|
| **Name** | Convex MCP |
| **Category** | Backend/Reactive |
| **Official** | Yes (Convex) |
| **Source** | https://github.com/get-convex/convex-mcp |
| **Transport** | stdio |
| **Install** | `npx @convex-dev/mcp-server` |

## Description
The Convex MCP Server connects AI agents to the Convex reactive backend platform. It enables running queries, mutations, and actions against Convex deployments, inspecting schemas and tables, and monitoring real-time logs — providing agents with a fully reactive, transactional database and serverless function runtime for building AI-powered applications.

## Key Tools
| Tool | Description |
|------|-------------|
| `run_query` | Execute a Convex query function |
| `run_mutation` | Execute a Convex mutation for data writes |
| `run_action` | Run a Convex action (external API calls) |
| `list_functions` | List all deployed Convex functions |
| `get_schema` | Retrieve the database schema definition |
| `list_tables` | List all tables in the database |
| `get_logs` | Retrieve function execution logs |

## Configuration
```json
{
  "mcpServers": {
    "convex": {
      "command": "npx",
      "args": ["@convex-dev/mcp-server"],
      "env": {
        "CONVEX_URL": "https://your-deployment.convex.cloud",
        "CONVEX_ADMIN_KEY": "your-admin-key"
      }
    }
  }
}
```

## Use Cases
1. Build reactive AI agent backends with real-time data synchronization
2. Store and query conversation histories with automatic indexing
3. Orchestrate multi-step agent workflows with transactional guarantees

## NVIDIA Skill Combinations
| Purpose | NVIDIA Skill | Reason |
|---------|-------------|--------|
| Reactive agent data | aiq-research | Store and sync research agent state in real-time |
| Knowledge persistence | aiq-research | Persist agent findings with reactive subscriptions |
| Workflow orchestration | aiq-research | Coordinate multi-agent research tasks transactionally |

## Prerequisites
- Convex account with an active deployment
- `CONVEX_URL` pointing to your deployment
- `CONVEX_ADMIN_KEY` for server-side access

## Security Notes
- Admin keys grant full access; never expose in client code
- Use Convex's built-in auth for user-scoped operations
- Monitor function logs for unexpected access patterns
- Rotate admin keys periodically through Convex dashboard

## References
- https://github.com/get-convex/convex-mcp
- https://docs.convex.dev/
- https://docs.convex.dev/api/
