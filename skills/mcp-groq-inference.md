# Groq LPU Inference MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | Groq LPU Inference MCP Server |
| **Category** | AI/Inference |
| **Official** | ⭐ Official / Verified |
| **Source** | https://github.com/groq/mcp-groq |
| **Transport** | stdio |
| **Install** | `npx -y @groq/mcp-server` |

## Tools & Capabilities
- `chat_completion` — Ultra-low latency LLM inference (Llama 3.3 70B, DeepSeek-R1-Distill)
- `transcribe_whisper` — Sub-second audio transcription using Whisper Large v3 on Groq LPU
- `list_models` — Inspect available open-weight models, context limits, and TPM quotas
- `estimate_latency` — Calculate estimated time-to-first-token (TTFT) and token generation speed
- `stream_tokens` — Stream response tokens at 500+ tokens/sec

## Client Configuration
```json
{
  "mcpServers": {
    "groq": {
      "command": "npx",
      "args": [
        "-y",
        "@groq/mcp-server"
      ],
      "env": {
        "GROQ_API_KEY": "YOUR_GROQ_API_KEY"
      }
    }
  }
}
```

## Security & Best Practices
- Enforce client request timeout and rate limiting. Guard against oversized prompt payloads.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
