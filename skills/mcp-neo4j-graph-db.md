# Neo4j Graph Database MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | Neo4j Graph Database MCP Server |
| **Category** | Database/Graph |
| **Official** | ⭐ Official (Neo4j) |
| **Source** | https://github.com/neo4j-contrib/mcp-neo4j |
| **Transport** | stdio |
| **Install** | `npx -y @neo4j/mcp-server` |

## Tools & Capabilities
- `get_schema` — Inspect node labels, relationship types, and property keys
- `read_cypher` — Execute read-only Cypher queries for relationship traversal
- `write_cypher` — Create or update graph nodes and edges with parameter binding
- `find_shortest_path` — Compute graph paths and community clusters between entities
- `validate_graph_integrity` — Detect orphaned nodes and enforce schema constraints

## Client Configuration
```json
{
  "mcpServers": {
    "neo4j": {
      "command": "npx",
      "args": [
        "-y",
        "@neo4j/mcp-server"
      ],
      "env": {
        "NEO4J_URI": "bolt://localhost:7687",
        "NEO4J_USERNAME": "neo4j",
        "NEO4J_PASSWORD": "password"
      }
    }
  }
}
```

## Security & Best Practices
- Enforce read-only Cypher mode by default. Parameterize all queries to prevent Cypher injection.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
