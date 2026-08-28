# Axiom MCP Server

## Overview
| Field | Value |
|-------|-------|
| **Name** | mcp-axiom |
| **Category** | Event Analytics |
| **Official** | Yes |
| **Source** | https://github.com/axiomhq/mcp |
| **Transport** | stdio |
| **Install** | `npx @axiomhq/mcp` |

## Description
Official Axiom MCP server for querying, ingesting, and analyzing event data at scale. Enables AI-powered exploration of datasets, APL query execution, event ingestion, and annotation management for observability and analytics workflows.

## Key Tools
| Tool | Description |
|------|-------------|
| `query` | Execute APL (Axiom Processing Language) queries |
| `list_datasets` | List all available datasets |
| `get_dataset_info` | Get schema and metadata for a dataset |
| `ingest_events` | Ingest structured events into a dataset |
| `create_annotation` | Create annotations for marking notable events |
| `get_virtual_fields` | Retrieve virtual field definitions |

## Configuration
```json
{
  "mcpServers": {
    "axiom": {
      "command": "npx",
      "args": ["@axiomhq/mcp"],
      "env": {
        "AXIOM_TOKEN": "your-api-token",
        "AXIOM_ORG_ID": "your-org-id"
      }
    }
  }
}
```

## Use Cases
1. Query and analyze AI agent execution traces and event timelines
2. Ingest inference pipeline telemetry events for performance analysis
3. Explore dataset schemas and create annotations for deployment milestones

## NVIDIA Skill Combinations
| Purpose | NVIDIA Skill | Reason |
|---------|-------------|--------|
| Trace agent events | aiq-research | Track and analyze AIQ agent execution events and research traces |

## Prerequisites
- Axiom account with active organization
- `AXIOM_TOKEN` — API token with query and ingest permissions
- `AXIOM_ORG_ID` — Organization identifier

## Security Notes
- Use scoped API tokens with minimal dataset access rather than personal tokens
- Rotate tokens regularly and avoid embedding them in source code

## References
- https://github.com/axiomhq/mcp
- https://axiom.co/docs/reference/api
