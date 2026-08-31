# GraphQL Introspection & Client MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | GraphQL Introspection & Client MCP Server |
| **Category** | Developer/GraphQL |
| **Official** | ⭐ Official / Verified |
| **Source** | https://github.com/graphql/mcp-graphql |
| **Transport** | stdio |
| **Install** | `npx -y @graphql/mcp-server` |

## Tools & Capabilities
- `introspect_schema` — Fetch complete GraphQL schema, types, queries, mutations, and directives
- `execute_query` — Execute parameterized GraphQL query against target endpoint
- `execute_mutation` — Execute GraphQL mutation with variables payload
- `validate_query` — Validate GraphQL query syntax and field selections against schema before execution
- `generate_query_scaffold` — Scaffold type-safe query templates for specific schema types

## Client Configuration
```json
{
  "mcpServers": {
    "graphql": {
      "command": "npx",
      "args": [
        "-y",
        "@graphql/mcp-server"
      ],
      "env": {
        "GRAPHQL_ENDPOINT": "https://api.example.com/graphql",
        "GRAPHQL_AUTH_HEADER": "Bearer YOUR_TOKEN"
      }
    }
  }
}
```

## Security & Best Practices
- Enforce query depth limits and query cost estimation to prevent denial of service.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
