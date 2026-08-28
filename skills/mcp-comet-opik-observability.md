# Comet Opik Observability MCP Server

## Overview
| Field | Value |
|-------|-------|
| **Name** | opik-mcp |
| **Category** | AI/ML |
| **Official** | Yes |
| **Source** | https://github.com/comet-ml/opik-mcp |
| **Transport** | stdio |
| **Install** | `npx @comet-ml/opik-mcp` |

## Description
Comet Opik MCP Server provides AI agents with access to LLM observability and evaluation data. It enables workflows to inspect traces, analyze span performance, compare experiments, and review feedback scores — giving developers full visibility into their AI application quality and behavior directly from the IDE without switching to a separate dashboard.

## Key Tools
| Tool | Description |
|------|-------------|
| `list_traces` | List recent traces with filtering options |
| `get_trace` | Get detailed trace data including all spans |
| `search_spans` | Search spans by name, status, or metadata |
| `get_metrics` | Retrieve aggregated metrics for a project |
| `list_experiments` | List evaluation experiments |
| `compare_experiments` | Compare metrics across experiments |
| `get_feedback_scores` | Get human/automated feedback scores for traces |

## Configuration
```json
{
  "mcpServers": {
    "opik": {
      "command": "npx",
      "args": ["@comet-ml/opik-mcp"],
      "env": {
        "OPIK_API_KEY": "your_api_key_here",
        "OPIK_WORKSPACE": "your_workspace"
      }
    }
  }
}
```

## Use Cases
1. Debug production LLM issues by inspecting traces and spans without leaving the IDE
2. Compare A/B experiment results across prompt versions or model configurations
3. Monitor feedback scores and quality metrics during iterative prompt engineering

## NVIDIA Skill Combinations
| Purpose | NVIDIA Skill | Reason |
|---------|-------------|--------|
| Observe RL training | nemo-rl-auto-research | Monitor reward signals and training quality in NeMo RL experiments |
| Monitor agent quality | aiq-research | Track AIQ agent trace quality and response accuracy over time |

## Prerequisites
- `OPIK_API_KEY` from your Comet account settings
- `OPIK_WORKSPACE` identifying your team workspace
- Node.js 18+ (for npx)

## Security Notes
- Traces may contain sensitive user inputs and model outputs; restrict workspace access
- API keys should be scoped to minimal required permissions

## References
- https://github.com/comet-ml/opik-mcp
- https://www.comet.com/docs/opik/
