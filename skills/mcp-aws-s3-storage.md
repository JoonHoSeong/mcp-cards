# AWS S3 Object Storage MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | AWS S3 Object Storage MCP Server |
| **Category** | Cloud/Storage |
| **Official** | ⭐ Official (AWS Labs) |
| **Source** | https://github.com/awslabs/mcp-s3 |
| **Transport** | stdio |
| **Install** | `npx -y @awslabs/mcp-s3` |

## Tools & Capabilities
- `list_buckets` — List all S3 buckets with creation dates and regions
- `list_objects` — Query object keys, prefixes, sizes, and storage classes
- `get_object_metadata` — Inspect content-type, ETag, versioning, and user tags
- `generate_presigned_url` — Create temporary secure download/upload presigned URLs
- `manage_bucket_policy` — Inspect CORS rules, lifecycle transitions, and bucket encryption

## Client Configuration
```json
{
  "mcpServers": {
    "aws-s3": {
      "command": "npx",
      "args": [
        "-y",
        "@awslabs/mcp-s3"
      ],
      "env": {
        "AWS_REGION": "us-west-2",
        "AWS_ACCESS_KEY_ID": "YOUR_AWS_ACCESS_KEY",
        "AWS_SECRET_ACCESS_KEY": "YOUR_AWS_SECRET_KEY"
      }
    }
  }
}
```

## Security & Best Practices
- Block public bucket creation. Never expose raw access keys in presigned URL generator logs.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
