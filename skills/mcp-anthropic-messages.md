# Anthropic Claude Messages MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | Anthropic Claude Messages MCP Server |
| **Category** | AI/LLM |
| **Official** | ⭐ Official (Anthropic Reference) |
| **Source** | https://github.com/modelcontextprotocol/servers/tree/main/src/anthropic |
| **Transport** | stdio |
| **Install** | `npx -y @modelcontextprotocol/server-anthropic` |

## Tools & Capabilities
- `create_message` — Generate completions using Claude 3.5 Sonnet / Claude 3.5 Haiku
- `batch_messages` — Submit asynchronous batch message jobs for 50% cost reduction
- `count_tokens` — Accurately count input/output tokens before dispatch
- `inspect_cache_status` — Audit prompt caching hit/miss efficiency metrics
- `evaluate_reasoning` — Enable extended thinking budget for complex planning tasks

## Client Configuration
```json
{
  "mcpServers": {
    "anthropic": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-anthropic"
      ],
      "env": {
        "ANTHROPIC_API_KEY": "YOUR_ANTHROPIC_API_KEY"
      }
    }
  }
}
```

## Security & Best Practices
- Never log raw prompt secrets. Ensure prompt caching headers do not leak tenant data.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
