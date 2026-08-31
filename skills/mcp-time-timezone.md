# Anthropic Time & Timezone MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | Anthropic Time & Timezone MCP Server |
| **Category** | Core/Time |
| **Official** | ⭐ Official (Anthropic Reference) |
| **Source** | https://github.com/modelcontextprotocol/servers/tree/main/src/time |
| **Transport** | stdio |
| **Install** | `npx -y @modelcontextprotocol/server-time` |

## Tools & Capabilities
- `get_current_time` — Retrieve current local or UTC timestamp in ISO 8601 format
- `convert_time` — Convert timestamps across IANA timezones with Daylight Saving Time (DST) calculation

## Client Configuration
```json
{
  "mcpServers": {
    "time": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-time"
      ]
    }
  }
}
```

## Security & Best Practices
- Pure stateless local date-time computation. Zero network overhead or permissions.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
