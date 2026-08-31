# Fireworks AI Low-Latency Inference MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | Fireworks AI Low-Latency Inference MCP Server |
| **Category** | AI/Inference |
| **Official** | ⭐ Official (Fireworks AI) |
| **Source** | https://github.com/fireworks-ai/mcp-fireworks |
| **Transport** | stdio |
| **Install** | `npx -y @fireworks-ai/mcp-server` |

## Tools & Capabilities
- `chat_completion` — Execute speculative decoding inference with 100+ tokens/sec throughput
- `deploy_lora_model` — Dynamically load fine-tuned LoRA adapters with zero server restart time
- `transcribe_audio` — Ultra-fast speech-to-text with Fireworks Whisper API
- `get_model_metrics` — Monitor Time-to-First-Token (TTFT) and decode latency

## Client Configuration
```json
{
  "mcpServers": {
    "fireworks": {
      "command": "npx",
      "args": [
        "-y",
        "@fireworks-ai/mcp-server"
      ],
      "env": {
        "FIREWORKS_API_KEY": "fw_..."
      }
    }
  }
}
```

## Security & Best Practices
- API Key token scoping with request budget monitoring.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
