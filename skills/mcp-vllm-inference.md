# vLLM Distributed Serving MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | vLLM Distributed Serving MCP Server |
| **Category** | AI/Inference |
| **Official** | ⭐ Official / Verified |
| **Source** | https://github.com/vllm-project/mcp-vllm |
| **Transport** | stdio |
| **Install** | `npx -y @vllm/mcp-server` |

## Tools & Capabilities
- `chat_completion` — High-throughput batched inference utilizing PagedAttention
- `inspect_engine_metrics` — Monitor GPU KV cache memory utilization, prefill/decode latency
- `manage_lora_adapters` — Dynamically activate, swap, and query fine-tuned LoRA adapters
- `tokenize_and_count` — Tokenize text and audit token allocation budget
- `stream_generation` — Stream tokens with sub-millisecond inter-token latency

## Client Configuration
```json
{
  "mcpServers": {
    "vllm": {
      "command": "npx",
      "args": [
        "-y",
        "@vllm/mcp-server"
      ],
      "env": {
        "VLLM_BASE_URL": "http://localhost:8000/v1",
        "VLLM_API_KEY": "1234"
      }
    }
  }
}
```

## Security & Best Practices
- Enforce max batch size and request queue depth to prevent GPU Out-of-Memory (OOM).

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
