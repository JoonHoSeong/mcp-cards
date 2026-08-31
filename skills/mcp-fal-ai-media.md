# fal.ai Generative Media MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | fal.ai Generative Media MCP Server |
| **Category** | AI/Media |
| **Official** | ⭐ Official (fal.ai) |
| **Source** | https://mcp.fal.ai/mcp |
| **Transport** | stdio |
| **Install** | `npx -y @fal-ai/mcp-server` |

## Tools & Capabilities
- `generate_media` — Trigger 1,000+ generative image, video, and audio models with sub-second latency
- `stream_video_generation` — Generate AI video clips using Kling, Runway, and LTX Video
- `query_model_schemas` — Inspect exact JSON input schemas and parameter types for fal pipelines
- `get_request_status` — Poll async generation progress and download result CDN URLs

## Client Configuration
```json
{
  "mcpServers": {
    "fal": {
      "command": "npx",
      "args": [
        "-y",
        "@fal-ai/mcp-server"
      ],
      "env": {
        "FAL_KEY": "YOUR_FAL_KEY"
      }
    }
  }
}
```

## Security & Best Practices
- Scoped FAL_KEY. Sanitize prompt text and enforce output resolution boundaries.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
