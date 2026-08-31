# OpenRouter Universal LLM Gateway MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | OpenRouter Universal LLM Gateway MCP Server |
| **Category** | AI/Gateway |
| **Official** | ⭐ Official (OpenRouter) |
| **Source** | https://mcp.openrouter.ai/mcp |
| **Transport** | stdio |
| **Install** | `npx -y @openrouter/mcp-server` |

## Tools & Capabilities
- `route_chat_completion` — Dispatch prompts to 400+ frontier & open-source LLMs dynamically
- `list_models_and_pricing` — Query real-time per-token pricing, context limits, and latency stats
- `compare_model_benchmarks` — Retrieve live Arena Elo rankings and throughput benchmarks
- `check_account_credits` — Query API balance, key expenditure caps, and usage statistics

## Client Configuration
```json
{
  "mcpServers": {
    "openrouter": {
      "command": "npx",
      "args": [
        "-y",
        "@openrouter/mcp-server"
      ],
      "env": {
        "OPENROUTER_API_KEY": "sk-or-v1-..."
      }
    }
  }
}
```

## Security & Best Practices
- Enforce maximum credit spend limits per API key.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
