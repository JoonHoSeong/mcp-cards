# DeepSeek AI Inference MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | DeepSeek AI Inference MCP Server |
| **Category** | AI/LLM |
| **Official** | ⭐ Official (DeepSeek) |
| **Source** | https://github.com/deepseek-ai/mcp-deepseek |
| **Transport** | stdio |
| **Install** | `npx -y @deepseek/mcp-server` |

## Tools & Capabilities
- `chat_completion` — Execute reasoning completions using DeepSeek-R1 and DeepSeek-V3
- `inspect_reasoning_content` — Parse and separate Chain-of-Thought thinking tokens from final answer
- `code_completion` — Specialized code generation and refactoring with DeepSeek-Coder
- `get_balance_and_usage` — Check API credit balances, token consumption, and rate limits
- `stream_reasoning_tokens` — Stream real-time thinking tokens during complex logic problem solving

## Client Configuration
```json
{
  "mcpServers": {
    "deepseek": {
      "command": "npx",
      "args": [
        "-y",
        "@deepseek/mcp-server"
      ],
      "env": {
        "DEEPSEEK_API_KEY": "sk-..."
      }
    }
  }
}
```

## Security & Best Practices
- Enforce model parameter temperature bounds. Mask sensitive prompt secrets.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
