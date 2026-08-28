# PostHog Product Analytics & Agent Tracking MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | PostHog Product Analytics & Agent Tracking MCP Server |
| **Category** | Analytics/Product |
| **Official** | ⭐ Official (PostHog) |
| **Source** | https://github.com/posthog/mcp-posthog |
| **Transport** | stdio |
| **Install** | `npx -y @posthog/mcp-server` |

## Tools & Capabilities
- `query_hogql` — Execute HogQL SQL queries across raw event streams, sessions, and user persons
- `inspect_feature_flags` — Query active feature flag evaluation status and rollout percentages
- `track_agent_tool_call` — Log AI agent tool usage, latency, and error rates via PostHog MCP Analytics
- `get_funnel_metrics` — Analyze multi-step conversion funnels and drop-off rates
- `list_insights_dashboards` — Retrieve saved analytical dashboards and trend charts

## Client Configuration
```json
{
  "mcpServers": {
    "posthog": {
      "command": "npx",
      "args": [
        "-y",
        "@posthog/mcp-server"
      ],
      "env": {
        "POSTHOG_API_KEY": "phx_...",
        "POSTHOG_HOST": "https://us.i.posthog.com"
      }
    }
  }
}
```

## Security & Best Practices
- Use Scoped Personal API Keys. Mask PII in event properties before ingestion.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
