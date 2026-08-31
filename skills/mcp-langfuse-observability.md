# Langfuse LLM Observability & Tracing MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | Langfuse LLM Observability & Tracing MCP Server |
| **Category** | AI/Observability |
| **Official** | ⭐ Official (Langfuse) |
| **Source** | https://github.com/langfuse/mcp-langfuse |
| **Transport** | stdio |
| **Install** | `npx -y @langfuse/mcp-server` |

## Tools & Capabilities
- `query_traces` — Search LLM execution traces, spans, latency, and cost per request
- `get_prompt_template` — Fetch active production prompt templates and compiled variables
- `create_prompt_version` — Deploy new prompt template version with commit message
- `log_evaluation_score` — Record automated or human evaluation scores on specific traces
- `get_cost_analytics` — Aggregate token expenditure and model performance metrics

## Client Configuration
```json
{
  "mcpServers": {
    "langfuse": {
      "command": "npx",
      "args": [
        "-y",
        "@langfuse/mcp-server"
      ],
      "env": {
        "LANGFUSE_PUBLIC_KEY": "pk-lf-...",
        "LANGFUSE_SECRET_KEY": "sk-lf-...",
        "LANGFUSE_HOST": "https://cloud.langfuse.com"
      }
    }
  }
}
```

## Security & Best Practices
- API Key authentication. Mask PII in trace payloads before dispatch.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
