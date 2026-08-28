# Naver Cloud Platform MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | Naver Cloud Platform MCP Server |
| **Category** | Cloud/Korea |
| **Official** | ⭐ Official (Naver Cloud) |
| **Source** | https://github.com/navercloud/mcp-ncp |
| **Transport** | stdio |
| **Install** | `npx -y @navercloud/mcp-server` |

## Tools & Capabilities
- `send_sms_sens` — Send SMS/LMS notifications via SENS (Simple & Easy Notification Service)
- `list_object_storage_buckets` — Query Naver Cloud Object Storage buckets and objects
- `call_clova_studio` — Invoke HyperCLOVA X model endpoint for Korean-specialized NLP reasoning
- `inspect_server_instances` — Query VPC VM instances, states, and private/public IPs
- `get_monitoring_metrics` — Retrieve CPU, memory, and network throughput metrics for cloud servers

## Client Configuration
```json
{
  "mcpServers": {
    "naver-cloud": {
      "command": "npx",
      "args": [
        "-y",
        "@navercloud/mcp-server"
      ],
      "env": {
        "NCP_ACCESS_KEY": "YOUR_NCP_ACCESS_KEY",
        "NCP_SECRET_KEY": "YOUR_NCP_SECRET_KEY"
      }
    }
  }
}
```

## Security & Best Practices
- HMAC-SHA256 request signature authentication. Restrict SENS SMS API rate limits.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
