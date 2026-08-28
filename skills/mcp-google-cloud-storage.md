# Google Cloud Storage MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | Google Cloud Storage MCP Server |
| **Category** | Cloud/Storage |
| **Official** | ⭐ Official (Google Cloud) |
| **Source** | https://github.com/google-cloud/mcp-gcs |
| **Transport** | stdio |
| **Install** | `npx -y @google-cloud/mcp-gcs` |

## Tools & Capabilities
- `list_buckets` — Query GCS buckets, storage classes, and lifecycle rules
- `list_objects` — Query BLOBs, prefixes, sizes, and update timestamps
- `upload_blob` — Upload local file or buffer to GCS bucket with custom metadata
- `download_blob` — Download GCS object to local file or memory stream
- `get_signed_url` — Generate time-limited V4 signed URLs for object sharing

## Client Configuration
```json
{
  "mcpServers": {
    "gcs": {
      "command": "npx",
      "args": [
        "-y",
        "@google-cloud/mcp-gcs"
      ],
      "env": {
        "GOOGLE_APPLICATION_CREDENTIALS": "/path/to/sa.json"
      }
    }
  }
}
```

## Security & Best Practices
- Enforce uniform bucket-level access. Block public bucket ACL modifications.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
