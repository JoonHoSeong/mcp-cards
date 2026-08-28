# Azure Blob Storage MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | Azure Blob Storage MCP Server |
| **Category** | Cloud/Storage |
| **Official** | ⭐ Official (Microsoft) |
| **Source** | https://github.com/azure/mcp-blob-storage |
| **Transport** | stdio |
| **Install** | `npx -y @azure/mcp-blob` |

## Tools & Capabilities
- `list_containers` — List Blob containers, access tiers, and metadata
- `list_blobs` — Query blobs with prefix filtering, tags, and content types
- `upload_blob` — Stream local files or data buffers to Azure Blob container
- `download_blob` — Read blob content and stream to local destination
- `generate_sas_token` — Generate Shared Access Signature (SAS) tokens with bounded expiry

## Client Configuration
```json
{
  "mcpServers": {
    "azure-blob": {
      "command": "npx",
      "args": [
        "-y",
        "@azure/mcp-blob"
      ],
      "env": {
        "AZURE_STORAGE_CONNECTION_STRING": "DefaultEndpointsProtocol=https;..."
      }
    }
  }
}
```

## Security & Best Practices
- Prefer User-Delegation SAS over Account Keys. Set short TTLs for generated download tokens.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
