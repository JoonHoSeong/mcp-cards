# Azure Cosmos DB MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | Azure Cosmos DB MCP Server |
| **Category** | Database/NoSQL |
| **Official** | ⭐ Official (Microsoft) |
| **Source** | https://github.com/azure/mcp-cosmosdb |
| **Transport** | stdio |
| **Install** | `npx -y @azure/mcp-cosmosdb` |

## Tools & Capabilities
- `list_databases` — Query Cosmos DB databases and provisioned throughput (RU/s)
- `list_containers` — Inspect partition keys, indexing policies, and container schemas
- `query_documents` — Execute SQL-syntax queries across JSON documents with partition scoping
- `upsert_document` — Insert or update single JSON document with partition key match
- `delete_document` — Remove document by ID and partition key

## Client Configuration
```json
{
  "mcpServers": {
    "azure-cosmosdb": {
      "command": "npx",
      "args": [
        "-y",
        "@azure/mcp-cosmosdb"
      ],
      "env": {
        "COSMOS_ENDPOINT": "https://your-account.documents.azure.com:443/",
        "COSMOS_KEY": "YOUR_COSMOS_KEY"
      }
    }
  }
}
```

## Security & Best Practices
- Always supply partition key in queries to prevent cross-partition RU consumption spikes.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
