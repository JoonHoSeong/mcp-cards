# AWS DynamoDB MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | AWS DynamoDB MCP Server |
| **Category** | Database/NoSQL |
| **Official** | ⭐ Official (AWS Labs) |
| **Source** | https://github.com/awslabs/mcp-dynamodb |
| **Transport** | stdio |
| **Install** | `npx -y @awslabs/mcp-dynamodb` |

## Tools & Capabilities
- `describe_table` — Inspect partition keys, sort keys, GSI, LSI, and billing mode
- `query_items` — Execute key condition queries with expression attributes and filters
- `scan_items` — Scan table with pagination and projected attributes
- `put_item` — Insert or replace single item with conditional writes
- `update_item` — Execute atomic update expressions on item attributes
- `validate_single_table` — Audit Single Table Design access patterns and key schemas

## Client Configuration
```json
{
  "mcpServers": {
    "aws-dynamodb": {
      "command": "npx",
      "args": [
        "-y",
        "@awslabs/mcp-dynamodb"
      ],
      "env": {
        "AWS_REGION": "ap-northeast-2",
        "AWS_ACCESS_KEY_ID": "YOUR_AWS_ACCESS_KEY",
        "AWS_SECRET_ACCESS_KEY": "YOUR_AWS_SECRET_KEY"
      }
    }
  }
}
```

## Security & Best Practices
- Enforce read-only mode in production. Require explicit limit on scan operations to prevent RCU spikes.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
