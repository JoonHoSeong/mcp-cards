# Zoom Meetings & Collaboration MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | Zoom Meetings & Collaboration MCP Server |
| **Category** | Communication/Video |
| **Official** | ⭐ Official (Zoom Video Communications) |
| **Source** | https://github.com/zoom/mcp-zoom |
| **Transport** | stdio |
| **Install** | `npx -y @zoom/mcp-server` |

## Tools & Capabilities
- `schedule_meeting` — Create scheduled Zoom video meeting with topic, passcode, and recurrence
- `get_meeting_transcript` — Retrieve AI-generated cloud recording audio transcripts
- `list_cloud_recordings` — Query downloadable video MP4/M4A recording files for past meetings
- `get_smart_summary` — Retrieve Zoom AI Companion smart meeting summaries, action items, and chapters

## Client Configuration
```json
{
  "mcpServers": {
    "zoom": {
      "command": "npx",
      "args": [
        "-y",
        "@zoom/mcp-server"
      ],
      "env": {
        "ZOOM_ACCOUNT_ID": "YOUR_ACCOUNT_ID",
        "ZOOM_CLIENT_ID": "YOUR_CLIENT_ID",
        "ZOOM_CLIENT_SECRET": "YOUR_CLIENT_SECRET"
      }
    }
  }
}
```

## Security & Best Practices
- OAuth 2.0 Server-to-Server app authentication. Mask attendee emails in public meeting logs.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
